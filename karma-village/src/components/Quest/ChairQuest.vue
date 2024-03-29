<template>
    <div>
        <navigation-component>
            <v-btn text @click="$router.go(-1)">
                <v-icon>arrow_back</v-icon>
            </v-btn>
            <v-btn text @click="$router.go(-1)" :disabled="!isQuestComplete">Complete Quest</v-btn>
        </navigation-component>
        
        <div>
            <v-card>
                <v-container>
                    <div>
                        <h2 class="quest-header">
                            Have a chair for me?<br/>
                        </h2>
                        <div class="quest-info">
                            <div>
                                <p>
                                    My ideal chair has these characteristics:
                                </p>
                                <ul>
                                    <li :style="{color: isChair}">Chair</li>
                                    <li :style="{color: isColorRed}">Color: red</li>
                                    <li :style="{color: getPlastic}">Made out of plastic</li>
                                </ul>
                            </div>
                            <div class="quest-image">
                                <v-img :src="require('@/assets/Red_Chair.png')"></v-img>
                            </div>
                        </div>          
                    </div>
                </v-container>
            </v-card>
            <v-card>
                <v-container>
                    <h3 class="quest-header">
                        Add a picture of the chair you can offer
                    </h3>
                    <div class="quest-uploader">
                        <div class="circle-loader" style="justify-self: center; display: none;margin-top: 24px;margin-bottom: 12px;" v-show="uploading">
                            <div class="checkmark draw"></div>
                        </div>
                        <div class="circle-loader load-complete" style="justify-self: center; display: none;margin-top: 24px;margin-bottom: 12px;" v-show="succesfulUpload">
                            <div class="checkmark draw"></div>
                        </div>
                    </div>
                    <div class="file-uploading" v-show="!uploading && !succesfulUpload">
                        <!--TODO: place a drop-zone element here, and call the function onFilesAdded whenever a file is added-->
                        <drop-zone :enabled="!uploading" @filesAdded="onFilesAdded"></drop-zone>
                        <div v-if="files">
                            <p v-for="(file, index) in files" :key="index">
                                {{file.name}}
                            </p>
                        </div>
                    </div>
                </v-container>
            </v-card>
        </div>
    </div>
</template>

<script lang="ts">
import DropZone from '@/components/Shared/DropZone.vue';
import NavigationComponent from '@/components/Shared/Navigation.vue';
import { googleApiService, GoogleVisionResponse } from '@/services/GoogleApi.service';
import JQuery from 'jquery';
import { Component, Prop, Vue } from 'vue-property-decorator';
import { IQuest } from '../../models/IQuest';
const $ = JQuery;
@Component({
    components: {DropZone, NavigationComponent},
})
export default class ChairQuest extends Vue {

    get getPlastic(): string {
        return this.getColorKeywordPresent('plastic');
    }
    get isColorRed() {
        return this.getColorKeywordPresent('red');
    }
    get isChair() {
        return this.getColorKeywordPresent('chair');
    }

    get isQuestComplete(): boolean {
        // TODO: return true when all keywords are
        return this.isKeywordPresent('chair')
            && this.isKeywordPresent('plastic')
            && this.isKeywordPresent('red');
    }

    @Prop() public quest!: IQuest;
    public keyWords: string[] = [];
    public files: File[] = [];
    private uploading = false;
    private succesfulUpload = false;

    public onFilesAdded(file: File[]) {
        this.uploading = true;
        this.succesfulUpload = false;

        this.files = file;
        const file1 = file[0];

        // TODO: do a google VISION API request and parse all the keywords (labelannotations -> .description) using the GoogleApiService (placed in the services package)
        googleApiService.detectLabels(file1).then((response: GoogleVisionResponse) => {
            this.keyWords = response.responses[0].labelAnnotations
                .map((labelAnnotation) => labelAnnotation.description);
            const dominantColor1 = response.responses[0].imagePropertiesAnnotation.dominantColors.colors.sort((color) => color.score)[0].color;
            const dominantColor = (dominantColor1.red > dominantColor1.blue && dominantColor1.red > dominantColor1.green) ? 'red' : 'doesntmatter';
            this.keyWords.push(dominantColor);
            response.responses[0].labelAnnotations.forEach(annotation => this.keyWords.push(annotation.description.toLowerCase()));
            this.uploading = false;
            if (this.isQuestComplete) {
                this.succesfulUpload = true;
                $('.checkmark').toggle();
            }
         }).catch(reason => {
             this.uploading = false;
             this.succesfulUpload = false;
             console.error('There was an error labeling the image through google api: ' + reason.toLocaleString());
             alert('An error occured while labeling the image through google vision api');
         });

        // setTimeout(t => {
        //     this.uploading = false;
        //     this.succesfulUpload = true;
        //     $('.checkmark').toggle();
        //     this.keyWords.push('red');
        //     this.keyWords.push('plastic');
        //     this.keyWords.push('chair');
        // }, 1000);
    }

    private getColorKeywordPresent(keyWord: string): string {
        // TODO: return the color green whenever the keyWord is present in the keywords field
        // make use of the isKeywordPresent function
        if (this.isKeywordPresent(keyWord)) {
            return 'green';
        }
        if (this.files.length !== 0) {
            return 'red';
        }
        return 'black';
    }

    private isKeywordPresent(keyword: string): boolean {
        // TODO: check if the keyword is present in the keywords field
        return this.files && this.keyWords && this.keyWords.indexOf(keyword) > -1;
    }
}
</script>

<style lang="scss" scoped>
    .quest-info {
        display: flex;
        flex-direction: row;
        justify-content: space-around;
        height: 100%;
    }

    .quest-uploader {
        display: flex;
        flex-direction: column;
        justify-content: space-evenly;
        align-items: center;
        height: 100%;
    }

    .quest-image{
        width: 40%;
    }

    ul {
        list-style: disc;
    }

    .file-select > .select-button {
        padding: 1rem;

        color: white;
        background-color: #2EA169;

        border-radius: .3rem;

        text-align: center;
        font-weight: bold;
    }

    /* Don't forget to hide the original file input! */
    .file-select > input[type="file"] {
        display: none;
    }

    // Define vars we'll be using
    $brand-success: #5cb85c;
    $loader-size: 7em;
    $check-height: $loader-size/2;
    $check-width: $check-height/2;
    $check-left: ($loader-size/6 + $loader-size/12);
    $check-thickness: 6px;
    $check-color: $brand-success;

    .circle-loader {
        margin-bottom: $loader-size/2;
        border: 3px solid rgba(0, 0, 0, 0.2);
        border-left-color: $check-color;
        animation: loader-spin 1.2s infinite linear;
        position: relative;
        display: inline-block;
        vertical-align: top;
        border-radius: 50%;
        width: $loader-size;
        height: $loader-size;
    }

    .load-complete {
        -webkit-animation: none;
        animation: none;
        border-color: $check-color;
        transition: border 500ms ease-out;
    }

    .checkmark {
        display: none;

        &.draw:after {
            animation-duration: 800ms;
            animation-timing-function: ease;
            animation-name: checkmark;
            transform: scaleX(-1) rotate(135deg);
        }

        &:after {
            opacity: 1;
            height: $check-height;
            width: $check-width;
            transform-origin: left top;
            border-right: $check-thickness solid $check-color;
            border-top: $check-thickness solid $check-color;
            content: '';
            left: $check-left;
            top: $check-height;
            position: absolute;
        }
    }

    @keyframes loader-spin {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    @keyframes checkmark {
        0% {
            height: 0;
            width: 0;
            opacity: 1;
        }
        20% {
            height: 0;
            width: $check-width;
            opacity: 1;
        }
        40% {
            height: $check-height;
            width: $check-width;
            opacity: 1;
        }
        100% {
            height: $check-height;
            width: $check-width;
            opacity: 1;
        }
    }
</style>
