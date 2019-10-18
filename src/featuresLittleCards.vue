<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="features-little-cards" :style="customStyle">
        <!-- wwManager:start -->
        <wwSectionEditMenu v-bind:sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <!-- Weweb Wallpaper -->
        <wwObject class="background" v-bind:ww-object="section.data.background" ww-category="background"></wwObject>
        <div class="content-container" :class="{'content-on-the-right': section.data.contentOntheRight}">
            <wwObject class="background rounded" :class="{'content-on-the-right': section.data.contentOntheRight}" v-bind:ww-object="section.data.background2" ww-category="background"></wwObject>
            <div class="left-content" :class="{'content-on-the-right': section.data.contentOntheRight}">
                <div class="card" v-for="(feature, index) in section.data.leftFeatures" :key="index">
                    <!-- wwManager:start -->
                    <div v-if="editMode" class="contextmenu-container">
                        <wwContextMenu class="contextmenu" tag="div" @ww-add-before="addFeature(index, 'before')" @ww-add-after="addFeature(index, 'after')" @ww-remove="removeFeature(index)">
                            <div class="wwi wwi-config"></div>
                        </wwContextMenu>
                    </div>
                    <!-- wwManager:end -->
                    <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="feature" class="list" @ww-add="add(feature, $event)" @ww-remove="remove(feature, $event)">
                        <wwObject tag="div" v-for="object in feature" :key="object.uniqueId" :ww-object="object"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>

            <div class="right-content" :class="{'content-on-the-right': section.data.contentOntheRight}">
                <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.rightFeatures" class="list" @ww-add="add(section.data.rightFeatures, $event)" @ww-remove="remove(section.data.rightFeatures, $event)">
                    <wwObject tag="div" v-for="object in section.data.rightFeatures" :key="object.uniqueId" :ww-object="object"></wwObject>
                </wwLayoutColumn>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            features: []
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
    created() {

        //Initialize section data
        let needUpdate = false
        this.section.data = this.section.data || {};
        this.features = this.section.data.features;

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

        if (needUpdate) {
            this.sectionCtrl.update(this.section);
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
            try {
                wwLib.wwPopups.addStory('SECTION_CONFIG', {
                    title: {
                        en: 'Section configuration',
                        fr: 'Configuration de la section'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'Content on the right side:',
                                    fr: 'Contenu sur le côté droit :'
                                },
                                type: 'radio',
                                key: 'contentOntheRight',
                                valueData: 'section.data.contentOntheRight'
                            }
                        ]
                    },
                    buttons: {
                        NEXT: {
                            text: {
                                en: 'Finish',
                                fr: 'Terminer'
                            },
                            next: null
                        }
                    }
                })
                let options = {
                    firstPage: 'SECTION_CONFIG',
                    data: {
                        section: this.section,
                    },
                }
                const result = await wwLib.wwPopups.open(options)
                console.log(result)
                let needUpdate = false
                if (typeof (result) != 'undefined') {
                    if (typeof (result.contentOntheRight) != 'undefined') {
                        this.section.data.contentOntheRight = result.contentOntheRight
                        needUpdate = true
                    }
                    if (needUpdate) {
                        this.sectionCtrl.update(this.section);
                        this.$forceUpdate();
                    }
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* wwManager:end */
        /* wwManager:end */
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang="scss" scoped>
.features-little-cards {
    position: relative;
    pointer-events: all;
    margin: 50px 0;
    .background {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        width: 100%;
        &.rounded {
            overflow: hidden;
            @media (min-width: 992px) {
                border-top-right-radius: 30px;
                border-bottom-right-radius: 30px;
                &.content-on-the-right {
                    border-top-right-radius: 0;
                    border-bottom-right-radius: 0;
                    border-top-left-radius: 30px;
                    border-bottom-left-radius: 30px;
                }
            }
        }
    }
    .content-container {
        position: relative;
        padding: 40px 0;
        width: 100%;
        display: flex;
        flex-direction: column;
        @media (min-width: 992px) {
            width: 90%;
            padding: 80px 0;
            border-top-right-radius: 30px;
            border-bottom-right-radius: 30px;
            flex-direction: row;
            &.content-on-the-right {
                margin-left: 10%;
                border-top-right-radius: 0;
                border-bottom-right-radius: 0;
                border-top-left-radius: 30px;
                border-bottom-left-radius: 30px;
            }
        }
        .left-content {
            position: relative;
            margin: 0 13%;
            flex-basis: 35%;
            display: flex;
            justify-content: flex-end;
            flex-direction: column;
            margin-bottom: -125px;
            order: 1;
            @media (min-width: 992px) {
                order: 0;
                &.content-on-the-right {
                    order: 1;
                    margin-left: 13%;
                }
            }
            .card {
                position: relative;
                width: 100%;
                background-color: white;
                border-radius: 15px;
                box-shadow: 0 2px 25px rgba(0, 0, 0, 0.15);
                margin-bottom: 20px;
                .contextmenu-container {
                    position: absolute;
                    top: 0;
                    left: 0;
                    width: 30px;
                    height: 30px;
                    transform: translate(-50%, -50%);
                    border-radius: 20px;
                    padding: 5px;
                    text-align: center;
                    background-color: #ef811a;
                    color: white;
                    .index {
                    }
                    .contextmenu {
                        font-size: 1.2rem;
                        cursor: pointer;
                        z-index: 2;
                    }
                }
            }
        }
        .right-content {
            position: relative;
            flex-basis: 26%;
            order: 0;
            display: flex;
            align-items: center;
            @media (min-width: 992px) {
                order: 1;
                margin-right: 13%;
                &.content-on-the-right {
                    order: 0;
                    margin-right: 0%;
                    margin-left: 13%;
                }
            }
            .list {
                width: 100%;
            }
        }
    }
}
</style>
