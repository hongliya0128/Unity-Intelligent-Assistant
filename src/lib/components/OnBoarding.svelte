<script>
	import { getContext } from 'svelte';
	const i18n = getContext('i18n');

	import ArrowRightCircle from './icons/ArrowRightCircle.svelte';

	export let show = true;
	export let getStartedHandler = () => {};
</script>

{#if show}
	<div class="unity-onboarding w-full h-screen max-h-[100dvh] text-white relative overflow-hidden">
		<div class="absolute inset-0 unity-grid"></div>
		<div class="absolute inset-0 unity-glow unity-glow-left"></div>
		<div class="absolute inset-0 unity-glow unity-glow-right"></div>

		<div class="fixed top-0 left-0 m-8 z-50">
			<div class="flex items-center gap-3 rounded-full border border-white/10 bg-slate-950/60 px-4 py-2 backdrop-blur-xl">
				<img
					src="/static/unity-assistant-mark.svg"
					class="w-7 h-7"
					alt={$i18n.t('Unity assistant logo')}
				/>
				<div class="text-left">
					<div class="text-sm font-semibold tracking-wide">Unity开发智能适配助手</div>
					<div class="text-[11px] uppercase tracking-[0.3em] text-cyan-300/80">
						Dev Console
					</div>
				</div>
			</div>
		</div>

		<div class="relative bg-transparent w-full h-screen max-h-[100dvh] flex z-10">
			<div class="flex flex-col justify-end w-full items-center pb-14 text-center px-6">
				<div class="max-w-5xl">
					<div class="inline-flex items-center gap-2 rounded-full border border-cyan-400/20 bg-cyan-400/10 px-4 py-2 text-xs uppercase tracking-[0.28em] text-cyan-200">
						Unity API / Bug Fix / Hotfix / Version Migration
					</div>

					<div class="mt-8 text-5xl lg:text-7xl font-secondary tracking-tight text-balance">
						{$i18n.t('Build faster.')}
						<br />
						{$i18n.t('Debug smarter.')}
					</div>

					<div class="mx-auto mt-6 max-w-3xl text-base leading-7 text-slate-300 lg:text-lg">
						{$i18n.t(
							'面向 Unity 项目开发的智能工作台，帮助你完成 API 查询、报错诊断、热更新转换与版本适配。'
						)}
					</div>

					<div class="mx-auto mt-8 grid max-w-4xl gap-3 text-left md:grid-cols-4">
						<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-4 backdrop-blur-md">
							<div class="text-xs uppercase tracking-[0.22em] text-cyan-200">API</div>
							<div class="mt-2 text-sm text-slate-200">官方文档检索</div>
						</div>
						<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-4 backdrop-blur-md">
							<div class="text-xs uppercase tracking-[0.22em] text-cyan-200">Debug</div>
							<div class="mt-2 text-sm text-slate-200">报错定位与修复</div>
						</div>
						<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-4 backdrop-blur-md">
							<div class="text-xs uppercase tracking-[0.22em] text-cyan-200">Hotfix</div>
							<div class="mt-2 text-sm text-slate-200">C# 转 xLua / toLua</div>
						</div>
						<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-4 backdrop-blur-md">
							<div class="text-xs uppercase tracking-[0.22em] text-cyan-200">Version</div>
							<div class="mt-2 text-sm text-slate-200">Unity 版本迁移建议</div>
						</div>
					</div>
				</div>

				<div class="flex justify-center mt-8">
					<div class="flex flex-col justify-center items-center">
						<button
							aria-label={$i18n.t('Get started')}
							class="relative z-20 flex p-1 rounded-full border border-cyan-400/30 bg-cyan-400/15 text-cyan-50 hover:bg-cyan-400/25 transition font-medium text-sm shadow-[0_0_30px_rgba(34,211,238,0.18)]"
							on:click={() => {
								getStartedHandler();
							}}
						>
							<ArrowRightCircle className="size-6" aria-hidden="true" />
						</button>
						<div class="mt-1.5 font-primary text-base font-medium" aria-hidden="true">
							{$i18n.t(`进入工作台`)}
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
{/if}

<style>
	.unity-onboarding {
		background:
			radial-gradient(circle at 20% 20%, rgba(34, 211, 238, 0.18), transparent 28%),
			radial-gradient(circle at 80% 18%, rgba(59, 130, 246, 0.14), transparent 30%),
			linear-gradient(160deg, #06121f 0%, #091521 48%, #02070d 100%);
	}

	.unity-grid {
		background-image:
			linear-gradient(rgba(148, 163, 184, 0.08) 1px, transparent 1px),
			linear-gradient(90deg, rgba(148, 163, 184, 0.08) 1px, transparent 1px);
		background-size: 44px 44px;
		mask-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.85), transparent);
	}

	.unity-glow {
		filter: blur(90px);
		opacity: 0.55;
	}

	.unity-glow-left {
		background: radial-gradient(circle, rgba(6, 182, 212, 0.35) 0%, transparent 50%);
		transform: translate(-20%, 12%);
	}

	.unity-glow-right {
		background: radial-gradient(circle, rgba(59, 130, 246, 0.3) 0%, transparent 52%);
		transform: translate(24%, -18%);
	}
</style>
