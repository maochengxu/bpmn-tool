<template>
    <div>
        <input type="file" accept=".bpmn" @change="handleFile" />
    </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
    name: 'ImportXml',
    emits: ['fileContent'], // declare the event
    setup(props, { emit }) {
        const handleFile = (event: Event) => {
            if (event.target) {
                const inputElement = event.target as HTMLInputElement
                if (inputElement.files && inputElement.files.length > 0) {
                    const file = inputElement.files[0]
                    const reader = new FileReader()
                    reader.onload = (e) => {
                        if (e.target) {
                            const result = e.target.result as string
                            // emit the event with the file content
                            emit('fileContent', result)
                        }
                    }
                    reader.readAsText(file)
                }
            }
        }
        return { handleFile }
    }
})
</script>
