<template>
    <div :class="['request', method.toLowerCase(), isOpened]">

        <div :class="['request-item']" @click="open = !open">
            <span class="method">{{method}}</span>
            <span class="path">{{url}}</span>
            <span class="description">{{description}}</span>   
        </div>
        <params-table :params="sourceList" v-show="open" />
    </div>
</template>

<script>
export default {
    props: {
        description: {
            type: String,
            default: ''
        },
        url: {
            type: String,
            default: '/'            
        },
        params: { 
            type: Array,
            default: () => { 
                return []
            }
        }, 
        headers: { 
            type: Array,
            default: () => { 
                return [] 
            }
        },         
        path: { 
            type: Array,
            default: () => { 
                return [] 
            }
        },      
        body: {
            type: Object,
            default: () => { 
                return { } 
            }
        },
        method: {
            type: String,
            default: 'GET'
        }
    },

    data () {
        return {
            open: false,
            apiObj: this.$parent.apiObj,
            dataHeaders: this.parseHeaders(this.headers),
            dataBody: this.parseBody(this.body),
            dataPath: this.parsePath(this.path),
            dataQuery: this.parseQuery(this.params)
        }
    },
    computed: {

        isOpened () {
            return this.open
        },

        host () {
            return this.$parent.host
        },

        sourceList () {
            return [].concat(this.dataPath, this.dataHeaders, this.dataBody, this.dataQuery)
        }

    },
    methods: {

        parsePath () {

            const paths = this.url.split('/').filter(it => {
                return it.indexOf('{') === 0 && it.indexOf('}') === (it.length - 1)
            }).map(it => {
                return it.replace(/\{/g, '').replace(/\}/g, '')
            })

            return paths.map(key => {
                
                let obj = this.path.filter(it => {
                    return it.key === key    
                })[0]

                if (!obj) {
                    obj = { key : key, value: '' }
                }

                if (typeof obj === 'string') {
                    obj = { key : key, value: obj}
                }

                return Object.assign({
                    source: 'path',
                    required: true,
                    type: 'string',
                    description: ''
                }, this.parseItems(obj))
            })
        },

        /**
         * request headers
         * 
         * {
         *  Authorization: 'Bearer {{access_token}}'
         * }
         * 
         * {{xxx}} 로 시작되는건 외부에서 입력받을 수 있는 변수로 대체됨 
         * 
         */
        parseHeaders (headers) {
            return headers.map(obj => {

                obj.params = this.parseHeaderValue(obj.description)

                return Object.assign({
                    source: 'header',
                    type: 'string',
                    required: true,
                    description: ''
                }, this.parseItems(obj))
            })
        },       
        /**
         * body 
         * 
         * {
         *  data : { a: 1, b: 2, c: 3 },
         *  contentType: 'application/json' 
         * }
         * 
         * or 
         * 
         * {
         *  data : "{ a: 1, b: 2, c: 3 }",
         *  contentType: 'text/plain' 
         * }
         * 
         */        
        parseBody (body) {

            if (!body.data) return [] 

            if (typeof body.data === 'string') {
                body.dataValue = body.data + "" 
            } else if (typeof body.data === 'object') {
                body.dataValue = JSON.stringify(body.data, null, 4)
            } 

            return [
                Object.assign({
                    source: 'body',
                    key: 'body',
                    contentType: 'application/json',
                    required: false,
                    description: ''
                }, body)
            ]

        },

        /**
         * query string 
         * 
         * [
         *   { key : 'xxx', value : 'xxx', type: 'string', defalut: 'xxx' , description : 'xxx', required: true },
         * ]
         * 
         */
        parseQuery (params) {
            return Object.keys(params).map(key => {
                
                let obj = params[key]

                if (typeof obj === 'string') {
                    obj = { key : key, value: obj  }
                }

                return Object.assign({
                    source: 'query',                    
                    type: 'string',
                    required: false,
                    description: ''
                }, this.parseItems(obj)) 
            })
        },        
        parseHeaderValue (headerValue) {
            const arr = headerValue.match(/\{\{(.*)\}\}/g)
            return arr.map(it => {
                let key = it.replace(/\{/g, '').replace(/\}/g, '')
                return {key, value: ''}
            })
        },
        parseItems (item) {
            if (!item.items) return item 

            item.items = (item.items || []).map( (it, index) => {

                if (typeof it === 'string' ) {
                    it = { text: it, value: it }
                }

                if (index === 0) {
                    it.selected = true 
                    item.inputValue = it.value 
                }

                return it
            })

            return item
        }
    }
}
</script>

<style scoped lang="stylus" type="text/css">
.request {
    border: 1px solid #000;
    border-radius: 4px;
    margin: 0 0 15px;

    .request-item {
        padding: 5px;
        cursor: pointer;
        display: flex;
        align-items: center;
    }

    &.post {
        border-color: #49cc90;
        background: rgba(73,204,144,.1);

        .method {
            background: #49cc90;
        }
    }

    &.put {
        border-color: #fca130;
        background: rgba(252,161,48,.1)

        .method {
            background: #fca130;
        }
    }

    &.get {
        border-color: #61affe;
        background: rgba(97,175,254,.1);

        .method {
            background: #61affe;
        }
    }    

    &.delete {
        border-color: #f93e3e;
        background: rgba(249,62,62,.1);

        .method {
            background: #f93e3e;
        }
    }

    .method {
        font-size: 14px;
        font-weight: 700;
        min-width: 80px;
        padding: 6px 15px;
        text-align: center;
        border-radius: 3px;
        background: #000;
        text-shadow: 0 1px 0 rgba(0,0,0,.1);
        font-family: sans-serif;
        color: #fff;
        text-transform: uppercase;
        box-sizing: border-box;
    }

    .path {
        font-size: 16px;
        display: flex;
        flex: 0 3 auto;
        align-items: center;
        word-break: break-all;
        padding: 0 10px;
        font-family: monospace;
        font-weight: 600;
        color: #3b4151;
        box-sizing: border-box;
    }

    .description {
        font-size: 13px;
        flex: 1;
        font-family: sans-serif;
        color: #3b4151;
    }
}
</style>
