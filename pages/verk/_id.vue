<template>
    <div class="colorlinks">
        <h2><span v-unsupported-chars="work.TitleSwedish"></span><span v-if="work.SubtitleSwedish">: {{work.SubtitleSwedish}}</span></h2>
        <work :work="work" :articles="articles"></work>
        <!-- <pre style="font-size:smaller">{{work | json}}</pre> -->
    </div>
</template>

<style lang="scss" scoped>
    .work {
        max-width : 500px;
    }
    h2 {
        max-width : 600px;
    }

</style>

<script>
    
    import backend from "assets/backend"
    import work from "~/components/work.vue"

    export default {
        name : "Work",
        head () {
            return {
                // title : this.article.id
            }
        },
        components : {
            work : work
        },
        async asyncData ({ params, error, payload }) {
            if(payload) {
                return { work : payload }
            }
            try {
                var {work, articles} = await backend.getWork(params.id)
                // console.log("work", article, work)
                return {work, articles}
            } catch (e) {
                console.error(e)
            }
        },
    }
</script>
