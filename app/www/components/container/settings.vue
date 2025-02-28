<template>
    <div class="content-within-page">
        <v-tour name="setting-tour" :steps="steps" :callbacks="tourCallbacks"></v-tour>
        <form class="form inner-form">
            <md-subheader class="md-primary">
                <b>{{ translated.USER }}</b>
            </md-subheader>
            <md-divider></md-divider>
            <md-field>
                <label for="lng">{{ translated.LANGUAGE }}</label>
                <md-select v-model="settings.lng" required class="v-step-1">
                    <md-option value="de">{{ translated.GERMAN }}</md-option>
                    <md-option value="en">{{ translated.ENGLISH }}</md-option>
                    <md-option value="es">{{ translated.SPANISH }}</md-option>
                </md-select>
            </md-field>
            <md-subheader class="md-primary">
                <b>{{ translated.CAR }}</b>
            </md-subheader>
            <md-divider></md-divider>
            <md-field>
                <label for="car">{{ translated.CAR }}</label>
                <md-select v-model="settings.car" required @md-selected="showCarMessage(); resetCapacity()" class="v-step-2">
                    <md-option value="IONIQ_BEV">{{ translated.IONIQ_BEV }}</md-option>
                    <md-option value="IONIQ_HEV">{{ translated.IONIQ_HEV }}</md-option>
                    <md-option value="IONIQ_PHEV">{{ translated.IONIQ_PHEV }}</md-option>
                    <md-option value="IONIQ_FL_EV">{{ translated.IONIQ_FL_EV }}</md-option>
                    <md-option value="IONIQ_5">{{ translated.IONIQ_5 }}</md-option>
                    <md-option value="SOUL_EV">{{ translated.SOUL_EV }}</md-option>
                    <md-option value="E_SOUL">{{ translated.E_SOUL }}</md-option>
                    <md-option value="AMPERA_E">{{ translated.AMPERA_E }}</md-option>
                    <md-option value="BOLT_EV">{{ translated.BOLT_EV }}</md-option>
                    <md-option value="KONA_EV">{{ translated.KONA_EV }}</md-option>
                    <md-option value="ZOE_Q210">{{ translated.ZOE_Q210 }}</md-option>
                    <md-option value="ZOE_ZE50">{{ translated.ZOE_ZE50 }}</md-option>
                    <md-option value="NIRO_EV">{{ translated.NIRO_EV }}</md-option>
                    <md-option value="ID_3">{{ translated.ID_3 }}</md-option>
                    <md-option value="ID_4">{{ translated.ID_4 }}</md-option>
                    <md-option value="ENYAQ_IV80">{{ translated.ENYAQ_IV80 }}</md-option>
                    <md-option value="E_GOLF">{{ translated.E_GOLF }}</md-option>
                </md-select>
                <span class="input-field-error">{{ carMessage }}</span>
            </md-field>
            <md-field>
                <label for="capacity">{{ translated.CAPACITY }}</label>
                <md-input v-model="settings.capacity" @input="settings.capacity = parseFloat($event || 0)" @blur="checkCapacity()"></md-input>
                <span class="md-suffix">kWh</span>
            </md-field>
            <md-field>
                <label for="consumption">{{ translated.CONSUMPTION }}</label>
                <md-input v-model="settings.consumption" type="number" placeholder="12.34" class="v-step-3" required></md-input>
                <span class="md-suffix">kWh/100km</span>
            </md-field>
            <md-subheader class="md-primary">
                <b>{{ translated.DEVICE }}</b>
            </md-subheader>
            <md-divider></md-divider>
            <md-field>
                <label for="devices">{{ translated.OBD2_DEVICE }}</label>
                <md-select v-model="settings.device" required class="v-step-4">
                    <md-option v-for="(device, index) in devices" :key="index" :value="device.id">{{ device.name }}</md-option>
                </md-select>
            </md-field>
            <a href="#" @click="showBluetoothSettings()">{{ translated.OBD2_DEVICE_PAIR }}</a>
            <md-subheader class="md-primary">
                <b>{{ translated.NOTIFICATIONS }}</b>
            </md-subheader>
            <md-divider></md-divider>
            <md-field>
                <input v-model.number="settings.soc" type="range" style="width: 100%">{{ settings.soc }}%
                <span class="md-helper-text v-step-5">{{ translated.SOC_THRESHOLD }}</span>
            </md-field>
            <br>
            <md-field>
                <label for="email">{{ translated.EMAIL }}</label>
                <md-input v-model="settings.email" type="email" placeholder="mail@example.com"></md-input>
            </md-field>
            <md-field>
                <img src="icons/notifications_active.svg" />
            <span>{{ translated.PUSH }}</span>
            <md-switch v-model="settings.push"></md-switch>
            </md-field>
        </form>
        <md-button class="md-raised md-primary" v-if="$route.path === '/register'" @click="$emit('settingsSaved', settings)">{{ translated.SAVE }}</md-button>
    </div>
</template>

<script>
    import translation from './../modules/translation.vue';
    import helper from './../modules/helper.vue';
    import storage from './../modules/storage.vue';
    import eventBus from './../modules/event.vue';

    export default {
        data() {
            return {
                translated: {},
                settings: {
                    soc: 70,
                    summary: false,
                    car: 'IONIQ_BEV',
                    capacity: 28,
                    lng: translation.getLocalLng()
                },
                carMessage: '',
                devices: [],
                autoboot: false,
                keepawake: false,
                steps: [{
                    target: '.v-step-1',
                    originalContent: 'TOUR_SETTING_1',
                    content: 'TOUR_SETTING_1'
                }, {
                    target: '.v-step-2',
                    originalContent: 'TOUR_SETTING_2',
                    content: 'TOUR_SETTING_2'
                }, {
                    target: '.v-step-3',
                    originalContent: 'TOUR_SETTING_3',
                    content: 'TOUR_SETTING_3'
                }, {
                    target: '.v-step-4',
                    originalContent: 'TOUR_SETTING_4',
                    content: 'TOUR_SETTING_4'
                }, {
                    target: '.v-step-5',
                    originalContent: 'TOUR_SETTING_5',
                    content: 'TOUR_SETTING_5'
                }],
                tourCallbacks: {
                    onStop: () => {
                        storage.setValue('tour_completed_setting', true);
                    }
                }
            };
        },
        components: {
            translation,
            storage
        },
        watch: {
            'settings.lng': 'setLng',
            'settings.push': 'setPush'
        },
        methods: {
            showCarMessage() {
                if (!this.settings.car) return this.carMessage = '';
                switch (this.settings.car) {
                    case 'IONIQ_HEV':
                    case 'IONIQ_PHEV':
                    case 'ZOE_Q210':
                    case 'AMPERA_E':
                    case 'BOLT_EV':
                        this.carMessage = translation.translate('CAR_BASIS_SUPPORT');
                        break;
                    case 'SOUL_EV':
                    case 'E_SOUL':
                    case 'IONIQ_FL_EV':
                    case 'IONIQ_5':
                    case 'NIRO_EV':
                        this.carMessage = translation.translate('CAR_INVALID_SUPPORT');
                        break;
                    case 'ID_3':
                    case 'ID_4':
                    case 'ENYAQ_IV80':
                    case 'ZOE_ZE50':
                        this.carMessage = translation.translate('CAR_EXPERIMENTAL_SUPPORT');
                        break;
                    default:
                        this.carMessage = '';
                        break;
                }
            },
            setLng() {
                storage.setValue('lng', this.settings.lng);
                this.translatePage();
                // translate tour
                // translate tour
                this.steps = this.steps.map(step => {
                    step.content = translation.translate(step.originalContent);
                    return step;
                });
                this.$emit('languageChanged');
                eventBus.$emit('settings_languageChanged');
            },
            setPush() {
                if (window.cordova && window.FCMPlugin) {
                    if (this.settings.push) FCMPlugin.subscribeToTopic(storage.getValue('token'));
                    else FCMPlugin.unsubscribeFromTopic(storage.getValue('token'));
                }
            },
            translatePage() {
                this.translated = translation.translatePage();
            },
            listDevices() {
                var self = this;

                bluetoothSerial.enable(enabled => {
                    bluetoothSerial.list(devices => {
                        self.devices = devices;
                    }, err => console.error(err));
                }, err => console.error(err));
            },
            showBluetoothSettings() {
                if (typeof bluetoothSerial !== 'undefined') bluetoothSerial.showBluetoothSettings();
            },
            checkCapacity() {
                this.settings.capacity = helper.getCarCapacity(this.settings.car, this.settings.capacity);
            },
            resetCapacity() {
                this.settings.capacity = 0;
                this.checkCapacity();
            }
        },
        created() {
            var self = this;

            self.translatePage();
            eventBus.$off('deviceReady');
            // wait for cordova device to be ready - apply listener, if not ready yet
            if(self.$root.deviceReady) self.listDevices();
            else {
                eventBus.$on('deviceReady', function() {
                    self.listDevices();
                });
            }
            self.settings = storage.getValue('settings', self.settings);
            // listen for save btn press (only for settings page) to push event
            eventBus.$off('settings_save');
            if (self.$route.path === '/settings') eventBus.$on('settings_save', () => self.$emit('settingsSaved', self.settings));
        },
        mounted() {
            var self = this;

            eventBus.$off('registration_accountCreated');
            eventBus.$on('registration_accountCreated', () => {
                if (!storage.getValue('tour_completed_setting')) {
                    // translate the tour and start afterwards
                    self.steps.forEach(step => step.content = translation.translate(step.content));
                    setTimeout(() => self.$tours['setting-tour'].start(), 1000);
                }
            });
        }
    }
</script>
