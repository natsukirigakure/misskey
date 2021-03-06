<template>
<iframe v-if="player" :src="player" heigth="250" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen />
<div v-else-if="tweetUrl && detail" class="twitter">
	<blockquote ref="tweet" class="twitter-tweet" :data-theme="$store.state.device.darkmode ? 'dark' : null">
		<a :href="url"></a>
	</blockquote>
</div>
<div v-else class="mk-url-preview">
	<a :href="url" target="_blank" :title="url" v-if="!fetching">
		<div class="thumbnail" v-if="thumbnail" :style="`background-image: url(${thumbnail})`"></div>
		<article>
			<header>
				<h1>{{ title }}</h1>
			</header>
			<p>{{ description }}</p>
			<footer>
				<img class="icon" v-if="icon" :src="icon"/>
				<p>{{ sitename }}</p>
			</footer>
		</article>
	</a>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import { url as misskeyUrl } from '../../../config';

export default Vue.extend({
	props: {
		url: {
			type: String,
			require: true
		},
		detail: {
			type: Boolean,
			required: false,
			default: false
		}
	},
	data() {
		return {
			fetching: true,
			title: null,
			description: null,
			thumbnail: null,
			icon: null,
			sitename: null,
			player: null,
			tweetUrl: null,
			misskeyUrl
		};
	},
	created() {
		const url = new URL(this.url);

		if (this.detail && url.hostname == 'twitter.com' && /^\/.+\/status(es)?\/\d+/.test(url.pathname)) {
			this.tweetUrl = url;
			const twttr = (window as any).twttr || {};
			const loadTweet = () => twttr.widgets.load(this.$refs.tweet);

			if (twttr.widgets) {
				Vue.nextTick(loadTweet);
			} else {
				const wjsId = 'twitter-wjs';
				if (!document.getElementById(wjsId)) {
					const head = document.getElementsByTagName('head')[0];
					const script = document.createElement('script');
					script.setAttribute('id', wjsId);
					script.setAttribute('src', 'https://platform.twitter.com/widgets.js');
					head.appendChild(script);
				}
				twttr.ready = loadTweet;
				(window as any).twttr = twttr;
			}
			return;
		}
		fetch('/url?url=' + encodeURIComponent(this.url)).then(res => {
			res.json().then(info => {
				if (info.url != null) {
					this.title = info.title;
					this.description = info.description;
					this.thumbnail = info.thumbnail;
					this.icon = info.icon;
					this.sitename = info.sitename;
					this.fetching = false;
					if ([ // THIS IS THE WHITELIST FOR THE EMBED PLAYER
						'afreecatv.com',
						'aparat.com',
						'applemusic.com',
						'amazon.com',
						'awa.fm',
						'bandcamp.com',
						'bbc.co.uk',
						'beatport.com',
						'bilibili.com',
						'boomstream.com',
						'breakers.tv',
						'cam4.com',
						'cavelis.net',
						'chaturbate.com',
						'cnn.com',
						'cybergame.tv',
						'dailymotion.com',
						'deezer.com',
						'djlive.pl',
						'e-onkyo.com',
						'eventials.com',
						'facebook.com',
						'fc2.com',
						'gameplank.tv',
						'goodgame.ru',
						'google.com',
						'hardtunes.com',
						'instagram.com',
						'johnnylooch.com',
						'kexp.org',
						'lahzenegar.com',
						'liveedu.tv',
						'livetube.cc',
						'livestream.com',
						'meridix.com',
						'mixcloud.com',
						'mixer.com',
						'mobcrush.com',
						'mylive.in.th',
						'myspace.com',
						'netflix.com',
						'newretrowave.com',
						'nhk.or.jp',
						'nicovideo.jp',
						'nico.ms',
						'noisetrade.com',
						'nood.tv',
						'npr.org',
						'openrec.tv',
						'pandora.com',
						'pandora.tv',
						'picarto.tv',
						'pscp.tv',
						'restream.io',
						'reverbnation.com',
						'sermonaudio.com',
						'smashcast.tv',
						'songkick.com',
						'soundcloud.com',
						'spinninrecords.com',
						'spotify.com',
						'stitcher.com',
						'stream.me',
						'switchboard.live',
						'tunein.com',
						'twitcasting.tv',
						'twitch.tv',
						'twitter.com',
						'vaughnlive.tv',
						'veoh.com',
						'vimeo.com',
						'watchpeoplecode.com',
						'web.tv',
						'youtube.com',
						'youtu.be'
					].some(x => x == url.hostname || url.hostname.endsWith(`.${x}`)))
						this.player = info.player;
				}	// info.url
			})	// json
		});	// fetch
	}	// created
});
</script>

<style lang="stylus" scoped>
iframe
	width 100%

root(isDark)
	> a
		display block
		font-size 14px
		border solid 1px isDark ? #191b1f : #eee
		border-radius 4px
		overflow hidden

		&:hover
			text-decoration none
			border-color isDark ? #4f5561 : #ddd

			> article > header > h1
				text-decoration underline

		> .thumbnail
			position absolute
			width 100px
			height 100%
			background-position center
			background-size cover

			& + article
				left 100px
				width calc(100% - 100px)

		> article
			padding 16px

			> header
				margin-bottom 8px

				> h1
					margin 0
					font-size 1em
					color isDark ? #d6dae0 : #555

			> p
				margin 0
				color isDark ? #a4aab3 : #777
				font-size 0.8em

			> footer
				margin-top 8px
				height 16px

				> img
					display inline-block
					width 16px
					height 16px
					margin-right 4px
					vertical-align top

				> p
					display inline-block
					margin 0
					color isDark ? #b0b4bf : #666
					font-size 0.8em
					line-height 16px
					vertical-align top

		@media (max-width 700px)
			> .thumbnail
				position relative
				width 100%
				height 100px

				& + article
					left 0
					width 100%

		@media (max-width 550px)
			font-size 12px

			> .thumbnail
				height 80px

			> article
				padding 12px

		@media (max-width 500px)
			font-size 10px

			> .thumbnail
				height 70px

			> article
				padding 8px

				> header
					margin-bottom 4px

				> footer
					margin-top 4px

					> img
						width 12px
						height 12px

.mk-url-preview[data-darkmode]
	root(true)

.mk-url-preview:not([data-darkmode])
	root(false)

</style>
