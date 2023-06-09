<template>
    <div class="containers">
        <div class="canvas" ref="mycanvas"></div>
        <!-- <div id="js-properties-panel" class="panel"></div> -->
        <properties-panel v-if="bpmnModeler" :modeler="bpmnModeler"></properties-panel>
        <div class="import-xml">
            <import-xml @fileContent="handleFileContent"></import-xml>
        </div>
        <ul class="buttons">
            <li>
                <a ref="saveDiagramLink" href="javascript:" title="SaveBpmn">Save BPMN</a>
            </li>
            <li>
                <a ref="saveSvgLink" href="javascript:" title="SaveSvg">Save SVG</a>
            </li>
        </ul>
    </div>
</template>

<script lang="ts">
import { ref, onMounted, defineComponent } from 'vue'
import BpmnModeler from 'bpmn-js/lib/Modeler'
import PropertiesPanel from './CustomPropertiesPanel.vue'
import ImportXml from './ImportExport/ImportXml.vue'
import xmlDefault from '../mocks/diagram_deafult'
// import { NButton } from 'naive-ui'
// import { xmlStr } from './defaultXml.ts'
// import {
//   BpmnPropertiesPanelModule,
//   BpmnPropertiesProviderModule
// } from 'bpmn-js-properties-panel'
// import camundaModdleDescriptors from 'camunda-bpmn-moddle/resources/camunda'

export default defineComponent({
    name: 'BpmnTool',
    components: {
        PropertiesPanel,
        ImportXml
        // NButton
    },

    setup() {
        const bpmnModeler: any = ref(null)
        const mycanvas: any = ref(null)
        const xmlContent = ref<string>('')
        const saveDiagramLink = ref(null)
        const saveSvgLink = ref(null)

        const init = () => {
            const canvasElement = mycanvas.value
            bpmnModeler.value = new BpmnModeler({
                container: canvasElement,
                keyboard: {
                    bindTo: document
                },
                // propertiesPanel: {
                //   parent: '#js-properties-panel'
                // },
                // additionalModules: [
                //   BpmnPropertiesProviderModule,
                //   BpmnPropertiesPanelModule,
                // ],
                // moddleExtensions: {
                //   camunda: camundaModdleDescriptors
                // }
            })
            if (xmlContent.value) {
                createNewDiagram(xmlContent.value)
            }
            else {
                createNewDiagram(xmlDefault)
            }
            success()
        }

        const handleFileContent = (content: string) => {
            xmlContent.value = content
            if (bpmnModeler.value) {
                createNewDiagram(xmlContent.value)
            }
        }

        const createNewDiagram = async (xmlStr: string) => {
            console.log('xml', xmlStr)
            try {
                const result = await bpmnModeler.value.importXML(xmlStr)
                const { warnings } = result
                console.log(warnings)
            } catch (err: any) {
                console.log(err.message, err.warnings)
            }
        }

        const success = () => {
            console.log('Success!')
            bpmnModeler.value.on('commandStack.changed', function () {
                saveDiagram(function (err: any, xml: any) {
                    console.log(xml)
                })
            })
            addBpmnListener()
        }

        const addBpmnListener = () => {
            bpmnModeler.value.on('commandStack.changed', function () {
                saveSvg(function (err: any, svg: any) {
                    setEncoded(saveSvgLink, 'diagram.svg', err ? null : svg)
                })
                saveDiagram(function (err: any, xml: any) {
                    setEncoded(saveDiagramLink, 'diagram.bpmn', err ? null : xml)
                })
                console.log('commandStack.changed')
            })
        }

        const saveDiagram = async (done: any) => {
            try {
                const result = await bpmnModeler.value.saveXML(
                    { format: true },
                    function (err: any, xml: any) {
                        done(err, xml)
                    }
                )
                const { xml } = result
                console.log(xml)
            } catch (err) {
                console.log(err)
            }
        }

        const saveSvg = async (done: any) => {
            try {
                const result = await bpmnModeler.value.saveSVG(done)
                const { svg } = result
                console.log(svg)
            } catch (err) {
                console.log(err)
            }
        }

        const setEncoded = (link: any, name: any, data: any) => {
            // Turn xml into uri
            const encodedData = encodeURIComponent(data)
            // Change <a> tag attributes
            // className is needed to trigger download
            // href is needed to download data
            // download is needed to name the file
            console.log(link, name, data)
            let xmlFile = new File([data], 'test.bpmn')
            console.log(xmlFile)
            if (data) {
                link.value.className = 'active'
                link.value.href = 'data:application/bpmn20-xml;charset=UTF-8,' + encodedData
                link.value.download = name
            }
        }

        onMounted(() => {
            init()
        })

        return {
            mycanvas,
            bpmnModeler,
            handleFileContent,
            saveDiagramLink,
            saveSvgLink
        }
    }
})
</script>

<style scoped>
.containers {
    background-color: #ffffff;
    width: 100%;
    height: calc(100vh - 52px);
}

.canvas {
    width: 100%;
    height: 100%;
}

.panel {
    position: absolute;
    right: 0;
    top: 0;
    width: 500px;
}

.buttons {
    position: absolute;
    left: 2vw;
    bottom: 30px;
    padding-left: 0;
}

.buttons li {
    display: inline-block;
    margin-right: 5px;
    padding-left: 0;
}

.buttons li a {
    color: #999;
    background: #eee;
    cursor: not-allowed;
    padding: 8px;
    border: 1px solid #ccc;
    text-decoration: none;
}

.buttons li a.active {
    color: #333;
    background: #fff;
    cursor: pointer;
}

.import-xml {
    position: absolute;
    left: 2vw;
    bottom: 85px;
}
</style>
