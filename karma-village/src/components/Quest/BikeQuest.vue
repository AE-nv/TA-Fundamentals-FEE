<template>
    <v-container>
        <div class="full-height">
            <navigation-component>
                <v-btn text @click="$router.go(-1)">
                    <v-icon>arrow_back</v-icon>
                </v-btn>
                <v-btn text @click="$router.go(-1)" :disabled="!isQuestComplete">Complete Quest</v-btn>
            </navigation-component>
            <div>
                <v-card>
                    <v-container>
                        <h2 class="quest-header">
                            Have a bike for me?
                        </h2>
                        <div class="quest-container">
                            <div class="quest-image">
                                <v-img :src="require('@/assets/bike.png')"></v-img>
                            </div>
                        </div>
                        <div class="quest-container">
                            <p>
                                I'm looking for a bike with two wheels, pedals and a saddle.
                                Preferably not too far from Leuven.
                                Leave a comment if you have one available.
                            </p>
                        </div>
                    </v-container>
                </v-card>
                <v-card>
                    <v-container>
                        <v-card  v-for="(res, index) in responses" :key="index" class="response-card">
                            {{ res }}
                        </v-card>
                        <text-area-box @text="addResponse"></text-area-box>
                    </v-container>
                </v-card>
            </div>
        </div>
    </v-container>
</template>

<script lang="ts">
import {Component, Vue} from 'vue-property-decorator';
import NavigationComponent from '@/components/Shared/Navigation.vue';
import TextAreaBox from '@/components/Shared/TextAreaBox.vue';

@Component({
    components: { NavigationComponent, TextAreaBox },
})
export default class BikeQuest extends Vue {
    private text: string = '';
    private responses: string[] = [];

    private get isQuestComplete(): boolean {
        return this.responses
            .some(res => res.indexOf('saddle') > -1 && res.indexOf('pedals') > -1 && res.indexOf('wheels') > -1);
    }

    private addResponse(value: string): void {
        this.responses.push(value);
    }

}
</script>

<style lang="scss" scoped>
    .quest-container {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .quest-image {
        width: 80%;
    }

    .response-card {
        margin: 0px 0px 10px 0px;
        padding: 10px;
    }
    
</style>
