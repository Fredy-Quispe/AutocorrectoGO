<template>
    <section class="file-download">

        <div class="document-preview">
            <div class="document-preview">
                <img :src="previewUrl" alt="Vista previa" class="img-document-preview" />
                <p class="filename">{{ resultFilename }}</p>
            </div>
        </div>

        <button class="download-button" @click="downloadResult">Descargar Documento</button>

        <div class="options">
            <div class="circular-buttons">
                <button class="option-button circular" @click="goBack(); deleteFile()" title="Eliminar"><font-awesome-icon
                        :icon="['fas', 'trash']" /></button>
            </div>
        </div>

        <footer class="footer">
            <button class="return-button" @click="goBack(); deleteFile()"> <font-awesome-icon :icon="['fas', 'left-long']"
                    class="return-ico" /> Volver a empezar</button>
        </footer>
    </section>
</template>
  
<script>
import axios from 'axios'
export default {
    name: 'FileDownload',
    data() {
        return {
            resultFilename: '',
            previewUrl: '',
        };
    },
    methods: {
        goBack() {
            console.log('Volviendo a FileUpload');
            this.$emit('go-Back');
        },
        showFileDownload(result) {

            if (result && result.pdf_result_filepath && result.vista_previa_filepath) {
                this.resultFilename = result.pdf_result_filepath;
                this.previewUrl = `http://localhost:5000/api/preview/${encodeURI(result.vista_previa_filepath.replace(/\\/g, '/'))}`;
                this.currentComponent = 'FileDownload';
            } else {
                console.error('Rutas de archivos no encontradas en la respuesta del servidor Download.');
            }
        },
        downloadResult() {
            console.log('Nombre del archivo resultante DVFT:', this.resultFilename);

            if (!this.resultFilename) {
                console.error('El nombre del archivo resultante no está definido DVFT.');
                return;
            }

            const encodedFilename = encodeURI(this.resultFilename.replace(/\\/g, '/'));
            const downloadUrl = `http://localhost:5000/api/download/${encodedFilename}`;
            console.log('URL de descarga:', downloadUrl);

            axios({
                url: downloadUrl,
                method: 'GET',
                responseType: 'blob',
            }).then(response => {
                const blob = new Blob([response.data], { type: 'application/pdf' });

                const url = window.URL.createObjectURL(blob);

                const link = document.createElement('a');
                link.href = url;
                link.download = 'output.pdf';
                document.body.appendChild(link);
                link.click();

                window.URL.revokeObjectURL(url);
                document.body.removeChild(link);
            }).catch(error => {
                console.error('Error al descargar el archivo', error);
            });
        },

        deleteFile() {
            if (!this.resultFilename) {
                console.error('El nombre del archivo resultante no está definido DVFT.');
                return;
            }

            const relativePreviewUrl = this.previewUrl.replace('http://localhost:5000/api/preview/', '');
            const encodedPreviewUrl = encodeURI(relativePreviewUrl.replace(/\\/g, '/'));
            const deleteUrl = `http://localhost:5000/api/delete/${encodedPreviewUrl}`;

            console.log('Contenido de relativePreviewUrl en FDFD ', relativePreviewUrl);
            console.log('Contenido de encodedPreviewUrl en FDFD', encodedPreviewUrl);
            console.log('Contenido de deleteUrl en FDFD', deleteUrl);

            axios({
                url: deleteUrl,
                method: 'DELETE',
            }).then(response => {
                console.log('Respuesta al eliminar resultante:', response.data.message);
                this.deleteFileResult();
            }).catch(error => {
                console.error('Error al eliminar el archivo', error);
            });
        },

        deleteFileResult() {
            if (!this.resultFilename) {
                console.error('La URL de la vista previa no está definida.');
                return;
            }

            const encodedFilename = encodeURI(this.resultFilename.replace(/\\/g, '/'));
            const deleteFileUrl = `http://localhost:5000/api/delete/${encodedFilename}`;

            console.log('Contenido de encodedFilename en FDFD2 ', encodedFilename);
            console.log('Contenido de deleteFileUrl en FDFD2 ', deleteFileUrl);


            axios({
                url: deleteFileUrl,
                method: 'DELETE',
            }).then(response => {
                console.log('Respuesta al eliminar el documento resultante:', response.data.message);
            }).catch(error => {
                console.error('Error al eliminar el documento resultante', error);
            });
        },
    },
};
</script>
  
<style scoped>
.file-download {
    margin-top: 104px;
}

.document-preview {
    width: 236px;
    height: 305px;
    border-radius: 5px;
    background-color: white;
    box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
    margin: auto;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
}

.img-document-preview {
    width: 200px;
    height: 300px;
}

.document-preview iframe {
    width: 100%;
    height: 100%;
}

.filename {
    font-size: 16px;
    color: #333;
    margin-top: 5px;
    text-align: center;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.download-button {
    width: 450px;
    height: 73px;
    border: none;
    border-radius: 50px;
    color: white;
    cursor: pointer;
    background-color: #4caf50;
    margin: 20px auto;
    display: flex;
    gap: 50px;
    align-items: center;
    justify-content: center;
    flex-direction: row;
    font-size: 20px;
    font-weight: 700;
    line-height: 26px;
    text-align: center;
}

.options {
    margin-top: 25px;
    text-align: center;
}

.option-button {
    background-color: white;
    border: 1px solid rgb(151, 151, 151);
    outline: 2px solid transparent;
    color: black;
    padding: 10px 20px;
    font-size: 20px;
    cursor: pointer;
    margin-right: 16px;
}

.option-button:hover {
    border-color: #45a049;
}

.circular-buttons {
    display: flex;
    justify-content: center;
}

.circular {
    border-radius: 50%;
    width: 45px;
    height: 45px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 14px;
}

.footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: white;
    width: 100%;
    height: 65px;
    position: fixed;
    bottom: 0;
    left: 0;
    z-index: 1000;
    box-shadow: 0px -2px 5px rgba(0, 0, 0, 0.1);
}

.return-button {
    width: 220px;
    height: 99%;
    background: white;
    border: none;
    margin-left: 0px;
    font-weight: 700;
    font-size: 16px;
    line-height: 19px;
    text-align: center;
    color: #313131;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

.return-button:hover {
    background-color: #45a049;
    color: white;
}

.return-ico {
    margin-right: 20px;
    color: magenta;
}

.return-button:hover .return-ico {
    color: white;
}
</style>
  