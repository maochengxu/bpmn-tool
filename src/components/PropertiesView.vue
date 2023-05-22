<template>
    <div class="custom-properties-panel">
        <div class="empty" v-if="selectedElements.length <= 0">Please choose one element</div>
        <div class="empty" v-else-if="selectedElements.length > 1">Only one element please</div>
        <div v-else>
            <fieldset class="element-item">
                <label>id</label>
                <span>{{ element.id }}</span>
            </fieldset>
            <fieldset class="element-item">
                <label>name</label>
                <input :value="element.name" @change="(event) => changeField(event, 'name')" />
            </fieldset>
            <fieldset class="element-item">
                <label>Info</label>
                <textarea
                    v-model="element.info"
                    @change="(event) => changeField(event, 'info')"
                    @click="(event) => parseMarkdown(event)"
                ></textarea>
                <div id="mdblock" v-html="markdownHtml"></div>
            </fieldset>
            <fieldset class="element-item">
                <label>Link</label>
                <input :value="element.link" @change="(event) => changeField(event, 'link')" />
            </fieldset>
            <fieldset class="element-item">
                <span class="link" @click="() => openLink(element.link)"> View </span>
            </fieldset>
        </div>
    </div>
</template>

<script lang="ts">
import { ref, onMounted, toRaw } from 'vue'
import MarkdownIt from 'markdown-it'

export default {
    name: 'PropertiesView',
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
                // console.log($attrs.info)
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
    right: 0;
    top: 0;
    width: 1000px;
    background-color: #ffffff;
    border-color: rgba(0, 0, 0, 0.09);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.09);
    padding: 20px;
}

#mdblock {
    text-align: center;
}
</style>
