<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="pricing-weweb-landing" :style="customStyle">
        <!-- wwManager:start -->
        <wwSectionEditMenu v-bind:sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <!-- Weweb Wallpaper -->
        <wwObject class="background" v-bind:ww-object="section.data.background" ww-category="background"></wwObject>
        <div class="content-block">
            <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.contentList" class="list" @ww-add="add(section.data.contentList, $event)" @ww-remove="remove(section.data.contentList, $event)">
                <wwObject tag="div" v-for="object in section.data.contentList" :key="object.uniqueId" :ww-object="object"></wwObject>
            </wwLayoutColumn>
        </div>
        <!-- SELECTOR -->
        <div v-if="plans && plans.length > 0" class="selector-container">
            <span class="interval-label" :class="{'selected': annually}">annually</span>
            <div class="selector" @click="toggleInterval">
                <div class="bubble" :class="{'right': !annually}"></div>
            </div>
            <span class="interval-label" :class="{'selected': !annually}">
                <span>monthly</span>
                <!-- wwManager:start -->
                <!-- UPDATE BUTTON -->
                <div v-if="editMode" class="update-button" @click="updateSectionData()">UPDATE</div>
                <!-- wwManager:end -->
            </span>
        </div>

        <!-- INITPLACEHOLDER -->
        <div v-if="!plans || plans.length == 0" class="placeholder">This section needs configuration to be displayed please click on the OPTIONS button</div>

        <!-- PLANS -->
        <div v-if="plans && plans.length > 0" class="plan-container">
            <div v-for="plan in plans" :key="plan.id" class="plan" :style="{'border': plan.border}">
                <div class="title" :style="{'color': plan.colorHash}">{{plan.title}}</div>
                <div class="price">{{(annually)? plan.annuallyPrice: plan.monthlyPrice}}€/month</div>
                <div class="pay-today" :class="{'visible': annually}">{{priceAnnuallyToPayToday(plan.annuallyPrice)}}€ to pay today</div>
                <div class="button" :style="{'background': plan.color}" center="true" invert="true" @click="selectPlan(plan.link)">
                    <span>SELECT</span>
                </div>
                <div v-for="(feature, index) in plan.features" :key="index" v-if="feature" class="feature" :class="{'ok': feature.icon != 'out', 'nok': feature.icon == 'out'}">
                    <div>
                        <i class="fas" :class="getIcon(feature.icon)"></i>
                    </div>
                    <div>{{getText(feature.label)}}</div>
                    <div class="help">
                        <i class="fas fa-question-circle"></i>
                        <div class="tooltip">
                            <div class="content">{{getText(feature.tooltip)}}</div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
// import { plans } from './globals.js'
/* wwManager:start */
import axios from 'axios'
wwLib.wwPopups.addStory('PRINCING_WEWEB_LANDING_CONFIG', {
    title: {
        en: 'Section config',
        fr: 'Configuration de la section'
    },
    type: 'wwPopupForm',
    storyData: {
        fields: [
            {
                label: {
                    en: 'Type of request:',
                    fr: 'Type de la requête :'
                },
                type: 'select',
                key: 'requestType',
                valueData: 'requestType',
                options: {
                    type: 'text',
                    values: [
                        {
                            value: 'get',
                            default: true,
                            text: {
                                en: 'GET',
                                fr: 'GET'
                            }
                        },
                        {
                            value: 'post',
                            text: {
                                en: 'POST',
                                fr: 'POST'
                            }
                        },
                        {
                            value: 'put',
                            text: {
                                en: 'PUT',
                                fr: 'PUT'
                            }
                        },
                        {
                            value: 'update',
                            text: {
                                en: 'UPDATE',
                                fr: 'UPDATE'
                            }
                        },
                        {
                            value: 'delete',
                            text: {
                                en: 'DELETE',
                                fr: 'DELETE'
                            }
                        }
                    ]
                }
            },
            {
                label: {
                    en: 'API endpoint to get the data:',
                    fr: 'Lien de l\'API pour récupérer les informations :'
                },
                type: 'text',
                key: 'planUrl',
                valueData: 'planUrl',
                desc: {
                    en: 'https://api.my-company/plans',
                    fr: 'Exemple : 0 10px 40px 0 rgba(113, 124, 137, 0.2)'
                }
            },
            {
                label: {
                    en: 'OR : Set the data yourself :',
                    fr: 'OU : Insérer les data vous-même :'
                },
                type: 'textarea',
                key: 'planJSON',
                valueData: 'planJSON',
                desc: {
                    en: 'This datas will fill in the section',
                    fr: 'Ces données vont compléter la section'
                },
                style: {
                    height: '600px'
                }
            }
        ]
    },
    buttons: {
        NEXT: {
            text: {
                en: 'Ok',
                fr: 'Ok'
            },
            next: false
        }
    }
})
/* wwManager:end */
export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            annually: true,
            plans: []
        }
    },
    computed: {
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },
        customStyle() {

            return {
                '--activeHeight': `${this.activeHeight + 28}px`
            }
        }

    },
    async created() {
        try {
            //Initialize section data
            let needUpdate = false
            this.section.data = this.section.data || {};
            this.features = this.section.data.features;
            this.plans = this.section.data.plans;
            // const response = await axios.get('http://localhost:1337/plans')
            // if (response && response.data) this.plans = response.data;
            console.log(this.plans)
            if (this.section.data.planJSON) {
                this.plans = JSON.parse(this.section.data.planJSON);
                console.log('planJSON', this.section.data.planJSON, 'plans', this.plans)
            }
            if (!this.section.data.background) {
                this.section.data.background = wwLib.wwObject.getDefault({
                    type: 'ww-color'
                });
                needUpdate = true
            }

            if (!this.section.data.background2) {
                this.section.data.background2 = wwLib.wwObject.getDefault({
                    type: 'ww-color'
                });
                needUpdate = true
            }

            if (!this.section.data.leftFeatures) {
                this.section.data.leftFeatures = [[wwLib.wwObject.getDefault({
                    type: 'ww-image'
                })]];
                needUpdate = true
            }

            if (!this.section.data.rightFeatures) {
                this.section.data.rightFeatures = [];
                needUpdate = true
            }

            if (!this.section.data.contentList) {
                this.section.data.contentList = [];
                needUpdate = true;
            }


            if (needUpdate) {
                this.sectionCtrl.update(this.section);
            }
        } catch (error) {
            console.log(error)
        }
    },
    mounted() {
    },
    beforeDestroy() {
        window.removeEventListener('scroll', this.onScroll)
    },
    beforeSave() {

    },
    methods: {
        toggleInterval() {
            this.annually = !this.annually;
        },
        priceAnnuallyToPayToday(priceAnnually) {
            const pa = priceAnnually * 12
            return pa
        },
        goToContactUs() {
            window.open('https://www.weweb.io/contact', '_blank');
        },
        getIcon(icon) {
            if (!icon) return {}
            let className = {}
            switch (icon) {
                case 'in':
                    className['fa-check'] = true;
                    break;
                case 'up':
                    className['fa-arrow-up'] = true;
                    break;
                case 'out':
                    className['fa-times'] = true;
                    break;

                default:
                    className['fa-times'] = true;
                    break;
            }
            return className
        },
        getText(text) {
            return wwLib.wwLang.getText(text)
        },
        selectPlan(link) {
            window.open(link, '_blank');
        },
        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newFeature = this.getNewFeature()
                this.section.data.leftFeatures.splice(index, 0, newFeature);
                this.sectionCtrl.update(this.section);
                // this.restartScrollListerner()
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        removeFeature(index) {
            try {
                this.section.data.leftFeatures.splice(index, 1);
                if (!this.section.data.leftFeatures.length) {
                    this.addFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
                this.restartScrollListerner()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        getNewFeature() {
            const newFeature = JSON.parse(JSON.stringify(this.section.data.leftFeatures[0]))
            wwLib.wwUtils.changeUniqueIds(newFeature)
            return newFeature
        },
        async openOptions() {
            let options = {
                firstPage: 'PRINCING_WEWEB_LANDING_CONFIG',
                data: {
                    requestType: this.section.data.requestType,
                    planUrl: this.section.data.planUrl,
                    planJSON: this.section.data.planJSON
                },
            }
            const result = await wwLib.wwPopups.open(options)
            console.log(result)
            let updateSection = false;
            if (result.requestType) {
                this.section.data.requestType = result.requestType;
                updateSection = true;
            }
            if (result.planUrl) {
                this.section.data.planUrl = result.planUrl;
                updateSection = true;
            }
            if (result.planJSON) {
                this.section.data.planJSON = result.planJSON;
                updateSection = true;
            }
            if (updateSection) {
                this.sectionCtrl.update(this.section);
                this.updateSectionData()
            }
        },
        async updateSectionData() {
            if (this.section.data.planJSON && JSON.parse(this.section.data.planJSON).length > 0) {
                this.plans = JSON.parse(this.section.data.planJSON)
            } else if (this.section.data.planUrl && this.section.data.requestType) {
                try {
                    const response = await axios({
                        method: this.section.data.requestType,
                        url: this.section.data.planUrl,
                        data: {}
                    });
                    this.plans = response.data
                    wwLib.wwNotification.open({
                        text: {
                            en: 'Section updated',
                            fr: 'Section mise à jour'
                        },
                        color: 'green',
                        duration: '5000'
                    })
                } catch (error) {
                    console.log(error);
                }
            }
        }
        /* wwManager:end */
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang="scss" scoped>
.pricing-weweb-landing {
    .background {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        width: 100%;
    }
    .content-block {
        width: 90%;
        margin: 0 5%;
        @media (min-width: 768px) {
            width: 70%;
            margin: 0 15%;
        }
        @media (min-width: 992px) {
            width: 60%;
            margin: 0 20%;
        }
    }
    .placeholder {
        height: 300px;
        background-color: #fafafa;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .selector-container {
        position: relative;
        pointer-events: all;
        padding-top: 50px;
        padding-bottom: 10px;
        display: flex;
        justify-content: center;
        align-items: center;
        & > * {
            z-index: 1;
        }
        .selector {
            position: relative;
            cursor: pointer;
            border: 1px solid white;
            background-color: white;
            border-radius: 20px;
            width: 55px;
            height: 26px;
            margin: 0 20px;
            .bubble {
                position: absolute;
                left: 5px;
                top: 50%;
                transform: translate(0, -50%);
                height: 15px;
                width: 15px;
                background-color: #e02a4d;
                border-radius: 100%;
                transition: all 0.4s ease;
                &.right {
                    left: 100%;
                    transform: translate(calc(-100% - 5px), -50%);
                }
            }
        }
        .interval-label {
            color: white;
            font-weight: bold;
            position: relative;
            &.selected {
                color: #e02a4d;
                text-shadow: 2px 0 0 #fff, -2px 0 0 #fff, 0 2px 0 #fff,
                    0 -2px 0 #fff, 1px 1px #fff, -1px -1px 0 #fff,
                    1px -1px 0 #fff, -1px 1px 0 #fff;
            }
            .update-button {
                position: absolute;
                background-color: white;
                border-radius: 20px;
                padding: 10px 30px;
                color: #e02a4d;
                top: 0;
                right: 0;
                transform: translate(calc(100% + 20px), -25%);
                cursor: pointer;
                &:hover {
                    background-color: #e02a4d;
                    color: white;
                }
            }
        }
    }
    .plan-container {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-wrap: wrap;
        & > * {
            z-index: 1;
        }
        .plan {
            background-color: white;
            margin-left: 30px;
            margin-top: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            border-radius: 10px;
            padding: 0 100px 30px 100px;
            .title {
                margin-top: 25px;
                font-size: 1.7rem;
                text-transform: capitalize;
            }
            .price {
                margin-top: 10px;
                color: #5e5e5e;
            }
            .button {
                cursor: pointer;
                pointer-events: all;
                margin: 25px 0 50px 0;
                width: 150px;
                text-align: center;
                color: white;
                padding: 5px 20px;
                border-radius: 20px;
            }
            .feature {
                display: flex;
                vertical-align: center;
                width: 100%;
                margin-bottom: 15px;
                color: #5e5e5e;
                i {
                    margin-right: 10px;
                    &.fa-check {
                        color: #cf2d7d;
                    }
                    &.fa-arrow-up {
                        color: #2e85c2;
                    }
                }
                &.nok {
                    color: #bbbbbb;
                }
                .help {
                    pointer-events: all;
                    position: relative;
                    padding-left: 10px;
                    transition: all 0.6s ease;
                    cursor: pointer;
                    .tooltip {
                        position: absolute;
                        display: none;
                        padding: 10px 10px;
                        background-color: white;
                        border-radius: 5px;
                        top: 0px;
                        left: 50%;
                        transform: translate(-50%, calc(-100% - 15px));
                        z-index: 2;
                        filter: drop-shadow(0px 0px 10px rgba(74, 74, 74, 0.2));
                        &:after {
                            content: "";
                            clip-path: polygon(50% 60%, 0 0, 100% 0);
                            z-index: 2;
                            position: absolute;
                            width: 15px;
                            height: 15px;
                            transform: translateX(-50%);
                            bottom: -15px;
                            left: 50%;
                            background-color: white;
                        }
                        .content {
                            width: 350px;
                        }
                    }
                    &:hover {
                        .tooltip {
                            display: block;
                        }
                    }
                }
            }
            .loader {
                margin-right: 5px;
            }
            .pay-today {
                color: #a7a7a7;
                font-style: italic;
                opacity: 0;
                transition: opacity 0.6s ease;
                &.visible {
                    opacity: 1;
                }
            }
        }
        .plan:first-child {
            margin-left: 0;
        }
    }
}
</style>
