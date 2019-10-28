<template>
    <div class="elem-captcha">
        <!-- wwManager:start -->
        <wwOrangeButton class="ww-orange-button" v-if="wwObjectCtrl.getSectionCtrl().getEditMode() == 'CONTENT'"></wwOrangeButton>
        <!-- wwManager:end -->
        <div v-if="!reload" class="g-recaptcha" :data-sitekey="wwObject.content.data.config.apiKey"></div>
    </div>
</template>

<script>
/* wwManager:start */
wwLib.wwPopups.addStory('WWFORM_CAPTCHA_OPTIONS', {
    title: {
        en: 'Edit Captcha',
        fr: 'Editer le Captcha'
    },
    type: 'wwPopupEditWwObject',
    buttons: null,
    storyData: {
        list: {
            CAPTCHA_CONFIG: {
                separator: {
                    en: 'Congiguration',
                    fr: 'Configuration'
                },
                title: {
                    en: 'Change captcha configuration',
                    fr: 'Changer la configuration du captcha'
                },
                desc: {
                    en: '',
                    fr: ''
                },
                icon: 'wwi wwi-config',
                shortcut: 'c',
                next: 'WWFORM_CAPTCHA_CONFIG'
            }
        }
    }
})

wwLib.wwPopups.addStory('WWFORM_CAPTCHA_CONFIG', {
    title: {
        en: 'Edit Captcha',
        fr: 'Editer le Captcha'
    },
    type: 'wwPopupForm',
    storyData: {
        fields: [
            {
                label: {
                    en: 'Google key :',
                    fr: 'Clé Google :'
                },
                desc: {
                    en: 'Google Api Key',
                    fr: `Clé de l'api Google`
                },
                type: 'text',
                key: 'apiKey',
                valueData: 'wwObject.content.data.config.apiKey',
            }
        ]
    },
    buttons: {
        OK: {
            text: {
                en: 'Ok',
                fr: 'Valider'
            },
            next: false
        }
    }
});
/* wwManager:end */

export default {
    name: "ww-form-captcha",
    props: {
        wwObjectCtrl: Object,
    },
    data() {
        return {
            wwLang: wwLib.wwLang,
            reload: false,
        }
    },
    computed: {
        wwObject() {
            return this.wwObjectCtrl.get();
        }
    },
    methods: {
        init() {
            const sc = document.createElement("script");
            sc.setAttribute('src', 'https://www.google.com/recaptcha/api.js');
            sc.setAttribute('type', 'text/javascript');
            sc.setAttribute('async', true);
            sc.setAttribute('defer', true);
            document.head.appendChild(sc);
            this.wwObject.content.data = this.wwObject.content.data || {}
            this.wwObject.content.data.config = this.wwObject.content.data.config || {}

            this.wwObjectCtrl.update(this.wwObject)
        },
        /* wwManager:start */
        async options() {
            let copyObj = JSON.parse(JSON.stringify(this.wwObject)) // to clean
            copyObj.uniqueId += 1
            let options = {
                firstPage: 'WWFORM_CAPTCHA_OPTIONS',
                data: {
                    wwObject: copyObj,
                }
            }
            try {
                const result = await wwLib.wwPopups.open(options);
                wwLib.wwObjectHover.setLock(this);
                /*=============================================m_ÔÔ_m=============================================\
                    CAPTCHA CONFIG
                \================================================================================================*/
                this.wwObject.content.data.config = this.wwObject.content.data.config || {};
                if (typeof (result) != 'undefined') {
                    if (typeof (result.apiKey) != 'undefined') {
                        this.wwObject.content.data.config.apiKey = result.apiKey;
                        this.wwObjectCtrl.update(this.wwObject);

                        this.reload = true;

                        setTimeout(() => {
                            this.reload = false;
                            this.$forceUpdate();

                            setTimeout(() => {
                                this.init();
                                this.$forceUpdate();
                            }, 100);
                        }, 100);
                    }
                }
            } catch (err) {
                wwLib.wwLog.error('ERROR', err)
            }
            wwLib.wwObjectHover.removeLock();
        }
        /* wwManager:end */
    },
    mounted() {
        this.init();
        this.$emit('ww-loaded', this);
    }
};
</script>

<style lang="scss" scoped>
.elem-captcha {
    width: 100%;
}
</style>
