<template>
    <div class="">
    <h2>Bibliografi – {{article}}</h2>
    

    <h3><div class="label">Språkurval och sortering</div> </h3>
    <select v-model="lang" @change="onLangChange(lang)">
      <option value="">Alla språk</option>
      <optgroup label="Originalspråk" >
        <option v-if="lang.LanguageName != 'Svenska'" :value="[lang.LanguageName, 'original']" v-for="lang in original">
            {{lang.LanguageName}}
        </option>
      </optgroup>
      <optgroup label="Källspråk">
        <option v-if="lang.LanguageName != 'Svenska'" :value="[lang.LanguageName, 'fran']" v-for="lang in source">
            {{lang.LanguageName}}
        </option>
      </optgroup>
      <optgroup label="Målspråk">
        <option v-if="lang.LanguageName != 'Svenska'" :value="[lang.LanguageName, 'till']" v-for="lang in target">
            {{lang.LanguageName}}
        </option>
      </optgroup>
    </select>

    <select class="sort" name="" id="" v-model="sortVal" @change="onSortChange(sortVal)">
        <option value="RealYear">År</option>
        <option value="Authors">Författare</option>
        <option value="TitleSwedish">Titel</option>
    </select>
    

    <!-- <a class="sc" :href="'/listor/avoversattare/' + $route.params.id" >Alla</a>
    
    <h3>
        <div class="label">Originalspråk</div>
    </h3>
    <ul class="lang-filter list">
        <li v-for="lang in original" class="list-item sc">
            <nuxt-link :to="'/listor/avoversattare/' + $route.params.id + '/original/' + lang.LanguageName">{{lang.LanguageName}}</nuxt-link>
        </li>
    </ul>
    
    <h3>
        <div class="label">Källspråk</div>
    </h3>
    <ul class="lang-filter list">
        <li v-for="lang in source" class="list-item sc">
            <nuxt-link :to="'/listor/avoversattare/' + $route.params.id + '/fran/' + lang.LanguageName">{{lang.LanguageName}}</nuxt-link>
        </li>
    </ul> -->

    <ul class="results colorlinks">
        <li v-for="item in connectionGroups" v-if="filterWorks(item.works).length">
            <h2 v-if="item.type == 2">Om {{ article }}</h2>
            <h2 v-else-if="item.type == 3">Skrifter av {{ article }}</h2>
            <h2 v-else-if="item.type == 4">Referenser</h2>
            <h2 v-else-if="item.type == 5">I artikeln omnämnda verk</h2>
            <ul v-if="[2,3,4,5].includes(item.type)">
                <li v-for="work in filterWorks(item.works)">
                    <work :work="work"></work>
                </li>
            </ul>            
            
            <template v-else-if="item.type == 1">
                <h2>Översättningar i bokform</h2>
                

                <div v-for="obj in biblTypeGroups">
                    <h3 v-if="obj.type != 1 && filterWorks(obj.works).length">
                        {{biblTypeData[String(obj.type)][0].BibliographyTypeName}}
                    </h3>
                    <ul>
                        <li v-for="work in filterWorks(obj.works)">
                            <work :work="work"></work>
                        </li>
                    
                    </ul>
                </div>
            </template>
            
        </li>
    </ul>

    </div>
</template>

<style lang="scss" scoped>
    h2 {
        margin-top: 1.8em;
    }
    .sort {
        margin-left: 2em;
    }
    .work {
        margin-bottom: 2em;
        max-width : 500px;
    }
    .list-inline-item:not(:last-child) {
        margin-right: 10px;
    }
    .label {
        // font-weight: bold;
        display : block;
        &:not(:first-of-type) {
            margin-top: 1em;
        }
    }
    
    .list-item {
            
    }

    .results {
        margin-top: 3em;
    }
</style>

<script>
    import backend from "assets/backend"
    import work from "~/components/work.vue"
    import _ from "lodash"

    import {naturalSort} from "assets/utils"
    function sortGroups(group, sortkey) {
        if(!group) {
            return
        }
        for(let {works} of group) {
            naturalSort(works, sortkey)
        }
    }


    export default {
        name : "AvOversattare",
        head () {
            return {
                title : "Verk för " + this.article
            }
        },
        components : {
            work : work
        },
        data() {
            return {
                sortVal : "RealYear",
                connectionGroups : [],
                lang : "",
                source : null,
                original : null,
                target: null,
                article : null,
                biblTypeGroups : null,
                biblTypeData : null
            }
        },
        async asyncData ({ params, error, route, from }) {

            console.log("asyncdata", from)
            let lang = ""
            if(params.lang) {
                lang = [params.lang, params.type]
            }
            let sortVal = route.query.sort || "RealYear"
            if(from && (from.matched[0].name == "listor-avoversattare-id" || from.matched[0].name == "avoversattare-filter")) {
                console.log("bailed asyncdata")
                return {lang, sortVal}
            }
            try{
                var backendData = await backend.getWorksByAuthor(params.id, sortVal)

            } catch(err) {
                console.log("Hittade ingen översättare vid det namnet.", err)
                error({ message: "Hittade ingen översättare vid det namnet.", statusCode: 404 })
            }

            sortGroups(backendData.connectionGroups, sortVal)
            sortGroups(backendData.biblTypeGroups, sortVal)

            return { ...backendData, lang, sortVal }
        },

        methods : {
            onLangChange : function([lang, type]) {
                if(lang) {
                    this.$router.push(`/listor/avoversattare/${this.$route.params.id}/${type}/${lang}`)
                } else {
                    this.$router.push(`/listor/avoversattare/${this.$route.params.id}`)
                    
                }

            },
            onSortChange : function(sortVal) {
                console.log("onSortChange", sortVal)
                this.$router.push({query : {sort: sortVal}})
                this.sortVal = sortVal

                sortGroups(this.connectionGroups, this.sortVal)
                sortGroups(this.biblTypeGroups, this.sortVal)
            },
            filterWorks : function(works) {
                if(!this.$route.params.lang) return works
                let type = this.$route.params.type
                let lang = this.$route.params.lang
                return works.filter((work) => {
                    if(type == "original") {
                        return lang == work.LanguageOriginalName
                    } else if(type == "fran") {
                        return (lang == work.LanguageSourceName) || (work.LanguageSourceName === null && lang == work.LanguageOriginalName)
                    } else if(type == "till") {
                        return lang == work.LanguageTargetName

                    }
                })
            },
        },
        computed : {
            langName : function() {
                return this.$route.query.l
            },
            langType : function() {
                return this.$route.query.lt
            },
            

        }
    }
</script>
