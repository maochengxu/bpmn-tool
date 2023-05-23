<template>
    <div class="custom-properties-panel">
        <div class="header">PROPERTIES PANEL</div>
        <div class="empty" v-if="selectedElements.length <= 0">Please choose one element</div>
        <div class="empty" v-else-if="selectedElements.length > 1">Only one element please</div>
        <div v-else>
            <div class="element-item">
                <div class="label">ID</div>
                <span>{{ element.id }}</span>
            </div>
            <div class="element-item">
                <div class="label">Name</div>
                <input :value="element.name" @change="(event) => changeField(event, 'name')" />
            </div>
            <div class="element-item">
                <div class="label">Information Markdown</div>
                <textarea
                    v-model="element.info"
                    @change="(event) => changeField(event, 'info')"
                    @click="(event) => parseMarkdown(event)"
                ></textarea>
            </div>
            <div class="element-item">
                <div class="label">Information Preview</div>
                <div class="preview" v-html="markdownHtml"></div>
            </div>
            <div class="element-item">
                <div class="label">Link</div>
                <input :value="element.link" @change="(event) => changeField(event, 'link')" />
                <button class="link" @click="() => openLink(element.link)" style="margin-top: 5px">
                    View
                </button>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { ref, onMounted, toRaw } from 'vue'
import MarkdownIt from 'markdown-it'

export default {
    name: 'PropertiesPanel',
    props: {
        modeler: {
            type: Object,
            default: () => ({})
        }
    },
    setup(props) {
        const selectedElements = ref([])
        const element = ref(null)
        const markdownHtml = ref('')

        onMounted(() => {
            init()
        })

        const init = () => {
            const { modeler } = props
            modeler.on('selection.changed', (e: any) => {
                selectedElements.value = e.newSelection
                element.value = e.newSelection[0]
                console.log(toRaw(element.value))
                setDefaultProperties()
                let md = new MarkdownIt()
                try {
                    let value = element.value['info']
                    if (value === undefined) {
                        value = ' '
                    }
                    markdownHtml.value = md.render(value)
                } catch (error) {
                    console.log(error)
                }
            })
            modeler.on('element.changed', (e: any) => {
                const { element: currentElement } = e
                if (!element.value) {
                    return
                }
                // update panel, if currently selected element changed
                if (currentElement.id === element.value.id) {
                    element.value = currentElement
                }
            })
        }

        const setDefaultProperties = () => {
            if (element.value) {
                const { businessObject } = element.value
                const { name, $attrs } = businessObject
                element.value['name'] = name
                element.value['link'] = $attrs.link
                element.value['info'] = $attrs.info
            }
        }

        const parseMarkdown = (event: any) => {
            let md = new MarkdownIt()
            const value = event.target.value
            markdownHtml.value = md.render(value)
        }

        const changeField = (event: any, type: string) => {
            const value = event.target.value
            let properties: any = {}
            properties[type] = value
            element.value[type] = value
            updateProperties(properties)
        }

        const updateProperties = (properties: any) => {
            const { modeler } = props
            const modeling = modeler.get('modeling')
            modeling.updateProperties(toRaw(element.value), properties)
        }

        const openLink = (link: string) => {
            window.open(link, '_blank')
        }

        return {
            selectedElements,
            element,
            markdownHtml,
            changeField,
            openLink,
            parseMarkdown
        }
    }
}
</script>

<style scoped>
.custom-properties-panel {
    position: absolute;
    right: 2vh;
    top: 2vh;
    width: 500px;
    max-height: 80vh;
    background-color: #ffffff;
    border-color: rgba(0, 0, 0, 0.09);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.327);
    padding: 20px;
    border-radius: 2%;
    overflow: auto;
}
.element-item {
    border-color: gainsboro;
    border-style: solid;
    border-width: 0px;
    padding: 10px;
    margin-top: -1px;
}

.preview {
    text-align: left;
    margin-top: 0px;
    padding-top: 0px;
    border: 1px solid gainsboro;
}

.label {
    font-size: 20px;
    font-weight: 400;
    text-align: left;
    color: #0e1a48;
}

input {
    border: 1px solid #ccc;
    border-radius: 3px;
    padding: 7px 0px;
    width: 475px;
    font-size: 16px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
}

.header {
    font-size: 25px;
    font-weight: 400;
    text-align: center;
    margin-bottom: 5px;
    font-family: museo-sans-1, museo-sans-2, 'Helvetica Neue', sans-serif;
    color: #0e1a48;
}

input:focus {
    border-color: #4d5dee;
    outline: 0;
    -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075), 0 0 8px rgba(102, 175, 233, 0.6);
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075), 0 0 8px rgba(102, 175, 233, 0.6);
}

textarea {
    border: 1px solid #ccc;
    border-radius: 3px;
    padding: 7px 0px;
    width: 475px;
    height: 100px;
    font-size: 16px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    resize: none;
}

textarea:focus {
    border-color: #4d5dee;
    outline: 0;
    -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075), 0 0 8px rgba(102, 175, 233, 0.6);
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075), 0 0 8px rgba(102, 175, 233, 0.6);
}

button {
    border: 1px solid #ccc;
    border-radius: 3px;
    padding: 7px 0px;
    width: 475px;
    font-size: 16px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
}
</style>
