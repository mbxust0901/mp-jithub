<template>
    <div class="repo-readme-wrap">
        <top-header title="README" :back="true"></top-header>
        <div class="readme-content">
            <rich-text v-if="!error" :nodes="nodes"></rich-text>   
            <empty :shown="error" :desc="msg"></empty>   
        </div>
    </div>
</template>

<script>    
    import { mapActions, mapGetters } from 'vuex';
    import MpvueMarkdownParser from 'mpvue-markdown-parser';

    import { Base64 } from 'js-base64';

    import empty from '@src/components/empty/index.vue';
    import topHeader from '@src/components/top-header/index.vue';

    export default {
        data () {
            return {
                error: false,
                loading: false,
                nodes: [],
                msg: '内容加载失败'
            };
        },

        components: {
            empty,
            topHeader
        },

        watch: {
            loading (val) {
                if (!val) {
                    wx.hideLoading();
                }
            }
        },

        methods: {
            ...mapActions([
                'getRepoReadme'
            ]),

            resetLoading () {
                this.loading = false;
            }
        },

        mounted () {
            const { username, reponame } = this.$root.$mp.query;
            wx.showLoading({
                title: '正在加载...',
                mask: true
            });

            this.loading = true;
            this.getRepoReadme({
                username,
                reponame
            }).then((res) => {
                this.loading = false;
                if (res.statusCode === 404) {
                    this.error = true;
                    this.msg = 'There is no README.md file in this repository';
                    return;
                }

                if (res.data.size > 35000) {
                    // 太大了 不渲染
                    this.error = true;
                    this.msg = `The README.md file is too large, visit https://github.com/${username}/${reponame} on pc, please.`;
                    return;
                }

                const md = Base64.decode(res.data.content);
                this.nodes = MpvueMarkdownParser(md);
            }).catch(e => {
                this.error = true;
                this.loading = false;
            });
        },

        onUnload () {
            this.nodes = [];
            this.error = false;
            this.loading = false;
        }
    };
</script>

<style>
    .readme-content {
        margin-top: 125rpx;
    }

    .readme-img {
        width: 100%;
    }
</style>