<script lang="ts">
	import { marked } from 'marked';
	import DOMPurify from 'dompurify';
	import { getContext, createEventDispatcher } from 'svelte';
	import { fade } from 'svelte/transition';

	const dispatch = createEventDispatcher();

	import { getChatList } from '$lib/apis/chats';
	import {
		user,
		models as _models,
		temporaryChatEnabled,
		selectedFolder,
		chats,
		currentChatPage
	} from '$lib/stores';
	import { sanitizeResponseContent } from '$lib/utils';
	import { WEBUI_API_BASE_URL } from '$lib/constants';

	import Suggestions from './Suggestions.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import EyeSlash from '$lib/components/icons/EyeSlash.svelte';
	import MessageInput from './MessageInput.svelte';
	import FolderPlaceholder from './Placeholder/FolderPlaceholder.svelte';
	import FolderTitle from './Placeholder/FolderTitle.svelte';

	const i18n = getContext('i18n');

	export let createMessagePair: Function;
	export let stopResponse: Function;

	export let autoScroll = false;
	export let atSelectedModel: Model | undefined;
	export let selectedModels: [''];
	export let history;
	export let prompt = '';
	export let files = [];
	export let messageInput = null;
	export let selectedToolIds = [];
	export let selectedFilterIds = [];
	export let pendingOAuthTools = [];
	export let showCommands = false;
	export let imageGenerationEnabled = false;
	export let codeInterpreterEnabled = false;
	export let webSearchEnabled = false;
	export let onUpload: Function = (e) => {};
	export let onSelect = (e) => {};
	export let onChange = (e) => {};
	export let toolServers = [];
	export let dragged = false;

	type WorkMode = 'free' | 'api' | 'debug' | 'hotfix' | 'migration';

	type PromptSuggestion = {
		title: [string, string];
		content: string;
	};

	type WorkModeConfig = {
		label: string;
		description: string;
		placeholder: string;
		prompt: string;
		suggestions: PromptSuggestion[];
	};

	let models = [];
	let selectedModelIdx = 0;
	let activeMode: WorkMode = 'free';
	let suggestionPrompts: PromptSuggestion[] = [];

	const modeContent: Record<WorkMode, WorkModeConfig> = {
		free: {
			label: '自由提问',
			description: '直接描述你的 Unity 开发问题，也可以任选下方场景卡片获得更有针对性的引导。',
			placeholder:
				'粘贴报错、代码片段、API 名称，或直接描述你的 Unity 开发问题...',
			prompt: '',
			suggestions: [
				{
					title: ['检查热更新写法', '把 C# 代码改成 xLua / toLua 兼容形式'],
					content: '请把这段 Unity C# 脚本改成 xLua 热更新写法，并指出需要额外导出的类型。'
				},
				{
					title: ['定位空引用异常', '根据报错栈分析 NullReferenceException 的根因'],
					content: '请帮我分析这个 NullReferenceException 报错，列出最可能的空对象来源和修复方法。'
				},
				{
					title: ['查询 Unity API', '说明方法作用、参数和适用场景'],
					content: '请解释一下 Addressables.LoadAssetAsync 的执行流程、常见报错和最佳实践。'
				}
			]
		},
		api: {
			label: 'API 查询',
			description: '查询 Unity API、参数含义和常见使用坑点',
			placeholder:
				'输入 API 名称、类名或使用场景，例如：CharacterController.Move 有哪些常见坑点？',
			prompt: '请解释一下 Unity CharacterController.Move 的使用方式、参数含义和常见坑点。',
			suggestions: [
				{
					title: ['查询 Unity API', '说明方法作用、参数和适用场景'],
					content: '请解释一下 Addressables.LoadAssetAsync 的执行流程、常见报错和最佳实践。'
				},
				{
					title: ['对比 API 用法', '说明新旧接口差异和替代方案'],
					content: '请对比 Resources.Load 和 Addressables.LoadAssetAsync 的适用场景与性能差异。'
				},
				{
					title: ['解释组件行为', '分析组件调用顺序和常见坑点'],
					content: '请解释 MonoBehaviour 生命周期函数的调用顺序，以及 Awake、OnEnable、Start 的区别。'
				}
			]
		},
		debug: {
			label: '报错调试',
			description: '定位异常原因并给出多种修复方案',
			placeholder:
				'粘贴报错栈、控制台输出或相关代码，例如：NullReferenceException 出现在 PlayerController.cs:54',
			prompt:
				'我遇到了 NullReferenceException，请根据报错栈帮我定位最可能为空的对象并给出修复方案。',
			suggestions: [
				{
					title: ['定位空引用异常', '根据报错栈分析 NullReferenceException 的根因'],
					content: '请帮我分析这个 NullReferenceException 报错，列出最可能的空对象来源和修复方法。'
				},
				{
					title: ['分析编译报错', '定位命名空间或类型缺失问题'],
					content: '请帮我分析 CS0246 编译错误，说明可能缺少的命名空间、程序集引用和修复步骤。'
				},
				{
					title: ['排查版本兼容问题', '检查 API 弃用和行为变化'],
					content: '这个 Unity 升级后的报错可能和 API 弃用有关，请帮我分析兼容性风险和替代方案。'
				}
			]
		},
		hotfix: {
			label: '热更新转换',
			description: '把 C# 逻辑转换成 xLua 或 toLua 写法',
			placeholder:
				'粘贴需要转换的 C# 代码，例如：请改成 xLua 热更新写法并说明导出配置',
			prompt: '请把这段 Unity C# 脚本改成 xLua 热更新写法，并指出需要额外导出的类型。',
			suggestions: [
				{
					title: ['检查热更新写法', '把 C# 代码改成 xLua / toLua 兼容形式'],
					content: '请把这段 Unity C# 脚本改成 xLua 热更新写法，并指出需要额外导出的类型。'
				},
				{
					title: ['分析绑定问题', '检查导出、委托和反射相关限制'],
					content: '请帮我分析这段热更新脚本里哪些类型、委托和事件需要在 xLua 中导出或适配。'
				},
				{
					title: ['补充迁移说明', '解释 C# 到 Lua 的关键改写思路'],
					content: '请把这段 C# 业务逻辑转换成 toLua 写法，并说明生命周期、调用方式和注意事项。'
				}
			]
		},
		migration: {
			label: '版本迁移',
			description: '比较 Unity 版本差异并评估兼容性风险',
			placeholder:
				'描述升级场景，例如：Unity 2022 LTS 升级到 Unity 6 后，需要优先检查哪些 API 和项目设置？',
			prompt: '请帮我整理从 Unity 2022 LTS 升级到 Unity 6 时最需要优先检查的 API 和项目设置。',
			suggestions: [
				{
					title: ['评估版本迁移', '对比 Unity 2022 LTS 与 Unity 6 的兼容性变化'],
					content: '请帮我整理从 Unity 2022 LTS 升级到 Unity 6 时最需要优先检查的 API 和项目设置。'
				},
				{
					title: ['迁移渲染管线', '检查 URP/HDRP 和材质兼容问题'],
					content: 'Unity 项目升级后渲染效果异常，请帮我分析 URP 升级过程中常见的材质和 Shader 兼容问题。'
				},
				{
					title: ['检查项目设置', '梳理输入系统、构建设置和插件适配'],
					content: '请列出 Unity 大版本迁移时，输入系统、构建设置和第三方插件最容易出问题的检查项。'
				}
			]
		}
	};

	$: if (selectedModels.length > 0) {
		selectedModelIdx = models.length - 1;
	}

	$: models = selectedModels.map((id) => $_models.find((m) => m.id === id));
	$: suggestionPrompts = modeContent[activeMode].suggestions;

	const selectMode = (mode: WorkMode) => {
		if (activeMode === mode) {
			activeMode = 'free';
			prompt = modeContent.free.prompt;
			onChange({ target: { value: prompt } });
			return;
		}

		activeMode = mode;
		prompt = modeContent[mode].prompt;
		onChange({ target: { value: prompt } });
	};
</script>

<div class="m-auto w-full max-w-6xl px-2 @2xl:px-20 translate-y-6 py-24 text-center">
	{#if $temporaryChatEnabled}
		<Tooltip
			content={$i18n.t("This chat won't appear in history and your messages will not be saved.")}
			className="w-full flex justify-center mb-0.5"
			placement="top"
		>
			<div class="flex items-center gap-2 text-gray-500 text-base my-2 w-fit">
				<EyeSlash strokeWidth="2.5" className="size-4" />{$i18n.t('Temporary Chat')}
			</div>
		</Tooltip>
	{/if}

	<div class="w-full text-3xl text-gray-800 dark:text-gray-100 text-center flex items-center gap-4 font-primary">
		<div class="w-full flex flex-col justify-center items-center">
			{#if $selectedFolder}
				<FolderTitle
					folder={$selectedFolder}
					onUpdate={async () => {
						await chats.set(await getChatList(localStorage.token, $currentChatPage));
						currentChatPage.set(1);
					}}
					onDelete={async () => {
						await chats.set(await getChatList(localStorage.token, $currentChatPage));
						currentChatPage.set(1);
						selectedFolder.set(null);
					}}
				/>
			{:else}
				<div class="flex flex-row justify-center gap-2.5 @sm:gap-3 w-fit px-5 max-w-xl">
					<div class="flex shrink-0 justify-center">
						<img
							src="/static/unity-assistant-mark.svg"
							class="size-9 @sm:size-10 rounded-full border border-slate-200 bg-white p-1 dark:border-slate-800 dark:bg-slate-900"
							alt="Unity开发智能适配助手"
							draggable="false"
						/>
					</div>

					<div class="text-3xl @sm:text-3xl line-clamp-1 flex items-center" in:fade={{ duration: 100 }}>
						{#if models[selectedModelIdx]?.name}
							<Tooltip
								content={models[selectedModelIdx]?.name}
								placement="top"
								className="flex items-center"
							>
								<span class="line-clamp-1">{models[selectedModelIdx]?.name}</span>
							</Tooltip>
						{:else}
							欢迎回来，{$user?.name}
						{/if}
					</div>
				</div>

				<div class="flex mt-1 mb-2">
					<div in:fade={{ duration: 100, delay: 50 }} class="flex flex-col items-center">
						<div class="mt-0.5 px-2 text-sm font-normal text-slate-500 dark:text-slate-400">
							当前模式：{modeContent[activeMode].label}
						</div>
						<div class="mt-1 px-2 text-sm font-normal text-gray-500 dark:text-gray-400 max-w-xl">
							{modeContent[activeMode].description}
						</div>
						{#if models[selectedModelIdx]?.info?.meta?.description ?? null}
							<Tooltip
								className="w-fit"
								content={DOMPurify.sanitize(
									marked.parse(
										sanitizeResponseContent(
											models[selectedModelIdx]?.info?.meta?.description ?? ''
										).replaceAll('\n', '<br>')
									)
								)}
								placement="top"
							>
								<div class="mt-1 px-2 text-sm font-normal text-gray-500 dark:text-gray-400 line-clamp-2 max-w-xl markdown">
									{@html DOMPurify.sanitize(
										marked.parse(
											sanitizeResponseContent(
												models[selectedModelIdx]?.info?.meta?.description ?? ''
											).replaceAll('\n', '<br>')
										)
									)}
								</div>
							</Tooltip>
						{/if}
					</div>
				</div>

				<div class="mb-6 mt-2 grid w-full max-w-4xl gap-3 text-left md:grid-cols-2 xl:grid-cols-4">
					{#each Object.entries(modeContent).filter(([mode]) => mode !== 'free') as [mode, action]}
						<button
							class="rounded-3xl border px-4 py-4 shadow-sm transition hover:-translate-y-0.5 hover:shadow-lg dark:bg-slate-900/80 {activeMode === mode
								? 'border-cyan-400 bg-cyan-50/80 shadow-cyan-100 dark:border-cyan-500 dark:bg-cyan-950/20'
								: 'border-slate-200 bg-white hover:border-cyan-300 dark:border-slate-800 dark:hover:border-cyan-500/60'}"
							on:click={() => selectMode(mode as WorkMode)}
						>
							<div class="text-[11px] uppercase tracking-[0.24em] text-cyan-600 dark:text-cyan-400">
								{action.label}
							</div>
							<div class="mt-2 text-sm leading-6 text-slate-600 dark:text-slate-300">
								{action.description}
							</div>
						</button>
					{/each}
				</div>
			{/if}

			<div class="text-base font-normal @md:max-w-3xl w-full py-3 {atSelectedModel ? 'mt-2' : ''}">
				<MessageInput
					bind:this={messageInput}
					{history}
					{selectedModels}
					bind:files
					bind:prompt
					bind:autoScroll
					bind:selectedToolIds
					bind:selectedFilterIds
					bind:imageGenerationEnabled
					bind:codeInterpreterEnabled
					bind:webSearchEnabled
					bind:atSelectedModel
					bind:showCommands
					bind:dragged
					{pendingOAuthTools}
					{toolServers}
					{stopResponse}
					{createMessagePair}
					placeholder={modeContent[activeMode].placeholder}
					{onChange}
					{onUpload}
					on:submit={(e) => {
						dispatch('submit', e.detail);
					}}
				/>
			</div>
		</div>
	</div>

	{#if $selectedFolder}
		<div
			class="mx-auto px-4 md:max-w-3xl md:px-6 font-primary min-h-62"
			in:fade={{ duration: 200, delay: 200 }}
		>
			<FolderPlaceholder folder={$selectedFolder} />
		</div>
	{:else}
		<div class="mx-auto max-w-2xl font-primary mt-2" in:fade={{ duration: 200, delay: 200 }}>
			<div class="mx-5">
				<Suggestions {suggestionPrompts} inputValue={prompt} {onSelect} />
			</div>
		</div>
	{/if}
</div>
