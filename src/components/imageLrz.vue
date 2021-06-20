<template>
    <div class="container">
        <div class="input-file">
            <span>请选择你要压缩的图片：</span>
            <input type="file" name="file" id="file" @change="onChange">
        </div>
        <div class="select-file-info">
            <div class="quality">
                <div class="titel">压缩率：</div>
                 <a-row class="content">
                    <a-col :span="12">
                        <a-slider v-model="quality" :min="0" :max="1" :step="0.1" @change="qualityChange"/>
                    </a-col>
                    <a-col :span="4">
                        <a-input-number
                            v-model="quality"
                            :min="0"
                            :max="1"
                            :step="0.1"
                            @change="qualityChange"
                            style="marginLeft: 16px"
                        />
                    </a-col>
                </a-row>
            </div>
            <div class="power">
                <div class="title">分辨率:（宽*高）</div>
                <div class="content">
                    <a-input placeholder="宽" v-model="endImgWidth" style="width: 80px" @change="widthChange"/> *
                    <a-input placeholder="高" v-model="endImgHeight" style="width: 80px" @change="heightChange"/>
                </div>
            </div>
        </div>
        <div class="file-info">
            <div class="start-file-info">
                <div class="title">压缩前</div>
                <div class="imgParams">
                    <div class="size">大小: {{ startSize }}</div>
                    <div class="powers">分辨率: {{ imgWidth }} * {{ imgHeight }}</div>
                </div>
                <div class="imgs" id="startImgSrc">
                    <img :src="startImgSrc" style="width: 100%">
                </div>
            </div>
            <div class="end-file-info">
                <div class="title">压缩后</div>
                <div class="imgParams">
                    <div class="size">大小: {{ endSize }}</div>
                    <div class="powers">分辨率: {{ endImgWidth }} * {{ endImgHeight }}</div>
                </div>
                <div class="imgs" id="endImgSrc">
                    <img :src="endImgSrc" style="width: 100%">
                </div>
                <div class="download">
                    <a-button type="primary" @click="downLoad" :disabled="endImgSrc ? false : true">
                        下载压缩图片
                    </a-button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import lrz from 'lrz'
export default {
    data() {
        return {
            quality: 0.6,
            imgWidth: 0,
            imgHeight: 0,
            startSize: 0,
            endImgSrc: '',
            startImgSrc: '',
            endSize: 0,
            endImgWidth: 0,
            endImgHeight: 0,
            fileName: '',
            power: 1,
            files: {}
        }
    },
    methods: {
        onChange(e) {
            const file = e.target.files[0]
            this.files = file
            this.fileName = file.name
            this.startSize = this.changSize(file.size)
            this.getFileInfo(file)
            setTimeout(() => {
                this.singleZip()
                console.log(file, this.fileName, this.imgWidth, this.imgHeight, this.power, '======file')
            }, 1000)
        },
        getFileInfo(imgFile) {
            const that = this
            let fileData = imgFile
            let reader = new FileReader();
            reader.onload = function (e) {
                const data = e.target.result;
                that.startImgSrc = data
                //加载图片获取图片真实宽度和高度
                let image = new Image();
                image.src = data;
                image.onload = function(){
                    that.imgWidth = image.width;
                    that.imgHeight = image.height;
                    that.endImgWidth = image.width
                    that.endImgHeight = image.height
                    that.power = image.width / image.height
                };
            }
            reader.readAsDataURL(fileData);
        },
        changSize(limit) {
            let size = "";
            if (limit < 0.1 * 1024) {
            if (limit < 0.1 * 100) {
                size = 0;
            } else {
                size = limit.toFixed(2) + "B" //小于0.1KB，则转化成B
            }

            } else if (limit < 0.1 * 1024 * 1024 * 1024) { //小于0.1MB，则转化成KB
                size = (limit / 1024).toFixed(2) + "KB"
            } else { //其他转化成GB
                size = (limit / (1024 * 1024 * 1024)).toFixed(2) + "GB"
            }
            const sizeStr = size + ""; //转成字符串
            const index = sizeStr.indexOf("."); //获取小数点处的索引
            const dou = sizeStr.substr(index + 1, 2) //获取小数点后两位的值
            if (dou == "00") { //判断后两位是否为00，如果是则删除00                
            return sizeStr.substring(0, index) + sizeStr.substr(index + 3, 2)
            }
            return size;
        },
        widthChange() {
            if(this.endImgWidth === 0) {
                this.endImgWidth = 0
                this.endImgHeight = 0
            } else {
                this.endImgHeight = parseInt(this.endImgWidth / this.power)
                this.singleZip()
            }     
        },
        heightChange() {
            if(this.endImgHeight === 0) {
                this.endImgWidth = 0
                this.endImgHeight = 0
            } else {
                this.endImgWidth = parseInt(this.endImgHeight * this.power)
                this.singleZip()
            }     
        },
        qualityChange() {
            this.singleZip()
        },
        singleZip() {
            const options = {
                quality: this.quality,
                width: this.endImgWidth,
                height: this.endImgHeight
            }
            lrz(this.files, options).then(res => {
                // 压缩后文件大小
                console.log(res)
                let img = new Image();
                img.src = res.base64; //将压缩后的base64赋给img的src,s然后可以赋给标签的src属性
                // let imgbase64 = res.base64
                this.endImgSrc = res.base64
                this.endSize = this.changSize(res.fileLen)
            });
        },
        downLoad() {
            const base = this.endImgSrc
            const name = this.fileName
            var a = document.createElement('a');
            a.download = name;
            a.innerHTML = 'download';
            a.href = base;
            a.click();
        }
    }
}
</script>

<style scoped lang="less">
    .container {
        width: 85%;
        height: 100vh;
        margin:  10px auto;
        font-size: 16px;
    }
    .input-file {
        width: 100%;
        height: 80px;
        padding-top: 20px;
    }

    .select-file-info {
        margin: 10px 0;
        width: 100%;
        height: 50px;
        border-block-end: 1px black dotted;

        .quality {
            width: 49%;
            height: 100%;
            display: flex;
            float: left;

            .title {
                flex: 1;
            }

            .content {
                flex: 4
            }
        }

        .power {
            width: 49%;
            height: 100%;
            display: flex;

            .title {
                flex: 1;
            }

            .content {
                flex: 2
            }
        }
    }

    .file-info {
        width: 100%;
        height: 550px;
        display: flex;

        .start-file-info {
            flex: 1;
            height: 100%;

            .title {
                width: 100%;
                line-height: 30px;
                text-align: center;
            }

            .imgParams {
                width: 100%;
                line-height: 30px;
                display: flex;
                text-align: center;

                .size {
                    flex: 1;
                }

                .powers {
                    flex: 1;
                }
            }

            .imgs {
                width: 90%;
                height: 450px;
                margin: 10px auto;
                border: 1px #f8f8f8 solid;
            }
        }

        .end-file-info {
            flex: 1;
            height: 100%;

            .title {
                width: 100%;
                line-height: 30px;
                text-align: center;
            }

            .imgParams {
                width: 100%;
                line-height: 30px;
                display: flex;
                text-align: center;

                .size {
                    flex: 1;
                }

                .powers {
                    flex: 1;
                }
            }

            .imgs {
                width: 90%;
                height: 450px;
                margin: 10px auto;
                border: 1px #f8f8f8 solid;
            }

            .download{
                width: 100%;
                text-align: right;
            }
        }
    }
</style>
