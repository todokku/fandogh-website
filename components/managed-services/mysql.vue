<template>

    <div>
        <div>
            <config-box :section-title="sections.service_config">

                <form class="col-xs-12" style="padding: 0">

                    <div style="display: flex">

                        <v-text-field
                                ref="name_selector"
                                style="font-family: iran-yekan;font-size: 1em; margin-left: -15px"
                                dir="ltr"
                                color="#0093ff"
                                required
                                :rules="[rules.counter, rules.required, rules.regex]"
                                v-model="manifest_model.name.name"
                                :label="manifest_model.name.label"
                                :counter="manifest_model.name.counter"
                                :maxlength="manifest_model.name.counter"
                                :hint="manifest_model.name.hint">

                        </v-text-field>

                        <popover :tooltip="tooltips.name"></popover>

                    </div>

                    <div style="display: flex; margin-top: 12px">

                        <v-text-field
                                style="font-family: iran-yekan;font-size: 1em; margin-left: -15px"
                                dir="ltr"
                                color="#0093ff"
                                required
                                :rules="[rules.counter, rules.required]"
                                v-model="mysql_manifest.password.value"
                                :label="password.label"
                                :hint="password.hint">

                        </v-text-field>

                        <popover :tooltip="tooltips.password"></popover>

                    </div>

                    <div style="display: flex; margin-top: 12px">

                        <v-text-field style="font-family: iran-yekan; font-size: 1em; margin-left: -15px"
                                      color="#0093ff"
                                      dir="ltr"
                                      prefix="Mi"
                                      :rules="[rules.default_memory, rules.required]"
                                      type="number"
                                      min="50"
                                      :hint="manifest_model.memory.hint"
                                      v-model.number="manifest_model.memory.amount"
                                      required
                                      @change="(manifest_model.memory.amount < 50 ? manifest_model.memory.amount = 200 : manifest_model.memory.amount)"
                                      :label="manifest_model.memory.label">

                        </v-text-field>


                        <popover :tooltip="tooltips.memory"></popover>


                    </div>

                </form>

            </config-box>

            <config-box :section-title="sections.phpmyadmin_config">

                <div style="width: 100%">


                    <div>
                        <span style="font-size: 0.8em; line-height: 1.75">
                            {{phpmyadmin_enabled.hint}}
                        </span>
                    </div>

                    <div style="margin-top: 16px">
                        <fan-checkbox
                                @click.native="phpMyAdminSelected"
                                v-tooltip="phpmyadmin.tooltip"
                                :object="phpmyadmin">

                        </fan-checkbox>
                    </div>
                </div>


            </config-box>

            <config-box :section-title="sections.storage_config">

                <div style="width: 100%">
                    <div>
                        <fan-checkbox v-for="(volume, index) in volumes"
                                      :key="index"
                                      @click.native="checkBoxSelected(index)"
                                      v-tooltip="volume.tooltip"
                                      :object="volume">

                        </fan-checkbox>
                    </div>

                    <div style="padding: 12px 0;">
                        <span style="font-size: 0.8em; line-height: 1.75">
                            {{selected_volume.tooltip}}
                        </span>
                    </div>

                    <div v-if="selected_volume.value === 'DedicatedVolume'" style="display: flex; margin-top: 12px">

                        <v-text-field ref="volume_name_selector"
                                      style="font-family: iran-yekan;font-size: 1em; margin-left: -15px"
                                      dir="ltr"
                                      color="#0093ff"
                                      :rules="[rules.volume_name_regex, rules.required]"
                                      v-model="mysql_manifest.volume_name.value"
                                      :label="volume_name.label"
                                      :hint="volume_name.hint">

                        </v-text-field>

                        <popover :tooltip="tooltips.volume_name"></popover>


                    </div>
                </div>

            </config-box>

        </div>


    </div>

</template>

<script>
    import ConfigBox from "../../components/wizard/box/config-box";
    import Popover from "../../components/wizard/tooltip/popover";
    import FanCheckbox from "../../components/wizard/select-box/fan-checkbox";

    export default {
        name: "mysql",
        layout: 'dashboard',
        components: {
            ConfigBox,
            Popover,
            FanCheckbox
        },
        props: {
            manifest_model: {
                type: Object,
                required: true
            }
        },
        model: {
            prop: 'manifest_model',
        },
        data() {
            return {
                selected_volume: {
                    label: "Shared Storage",
                    value: "SharedStorage",
                    selected: true,
                    tooltip: 'داده‌های شما در پوشه‌ای با نام سرویس فعلی شما در Shared Storage ذخیره خواهند شد.'
                },
                phpmyadmin: {
                    label: "PhpMyAdmin",
                    value: "PhpMyAdmin",
                    selected: true,
                    tooltip: 'با استفاده از این دکمه می‌توانید تعیین کنید رابط ادمین PhpMyAdmin ساخته شود یا خیر'
                },
                volumes: [
                    {
                        label: "Shared Storage",
                        value: "SharedStorage",
                        selected: true,
                        tooltip: 'داده‌های شما در پوشه‌ای با نام سرویس فعلی شما در Shared Storage ذخیره خواهند شد.'
                    },
                    {
                        label: "Dedicated Volume",
                        value: "DedicatedVolume",
                        selected: false,
                        tooltip: 'داده‌های شما در پوشه‌ای با نام سرویس فعلی شما در Dedicated Volume مورد نظر شما ذخیره خواهند شد.'
                    }
                ],
                password: {
                    label: 'رمز عبور database',
                    hint: 'رمز عبور به صورت پیش فرض root است',
                    name: ''

                },
                phpmyadmin_enabled: {
                    label: 'PhpMyAdmin رابط ادمین',
                    hint: 'در صورتی که نیاز دارید از رابط ادمین PhpMyAdmin استفاده کنید، آن را فعال نمایید. این قابلیت به صورت پیش‌فرض انتخاب برای شما ساخته خواهد شد.',
                    name: ''
                },
                volume_name: {
                    label: 'نام Volume',
                    hint: 'نام volume که قصد دارید داده‌های شما در آن ذخیره شوند',
                    name: ''
                },
                mysql_manifest: {
                    password: {
                        name: 'mysql_root_password',
                        value: ''
                    },
                    phpmyadmin_enabled: {
                        name: 'phpmyadmin_enabled',
                        value: true
                    },
                    volume_name: {
                        name: 'volume_name',
                        value: null
                    }
                },
                rules: {
                    required: value => !!value || 'پر کردن این فیلد اجباری‌ است',
                    counter: value => value.length <= 100 || 'مقدار وارد شده نباید بیش از ۱۰۰ کاراکتر باشد',
                    default_memory: value => value >= 50 || 'کمترین میزان رم قابل قبول ۵۰ مگابایت است',
                    regex: value => new RegExp('^[a-z]+(-*[a-z0-9]+)*$').test(value) || 'نام وارد شده صحیح نمی‌باشد (تنها ترکیب حروف کوچک a تا z، اعداد و خط تیره (-) معتبر هستند)',
                    default_replica: value => value >= 1 || 'کمترین مقدار مجاز ۱ است',
                    valid_port: value => value >= 1 && value <= 65535 || 'مقدار پورت باید بین ۱ تا ۶۵۵۳۵ باشد',
                    volume_name_regex: value => new RegExp('^[a-z]+(-*[a-z0-9]+)*$').test(value) || 'نام وارد شده صحیح نمی‌باشد (تنها ترکیب حروف کوچک a تا z، اعداد و خط تیره (-) معتبر هستند)',
                },
                tooltips: {
                    name: {
                        title: 'نام سرویس',
                        text: 'نامی که برای سرویس خود انتخاب می‌کنید، این نامی است که باقی سرویس‌ها از طریق آن٬ این سرویس را پیدا می‌کنند. این مورد مشابه سوئیچ name-- در دستور fandogh service deploy است.',
                        url: 'https://docs.fandogh.cloud/docs/service-manifest.html#name'
                    },
                    memory: {
                        title: 'تعیین رم سرویس',
                        text: 'از طریق این فیلد می‌توانید مشخص کنید که سرویس شما چه مقدار منابع در اختیار داشته باشد.',
                        url: 'https://docs.fandogh.cloud/docs/service-manifest.html#resources'
                    },
                    volume_name: {
                        title: 'Volume Name',
                        text: 'نام Dedicated Volume که قصد دارید داده‌ها در آن ذخیره شوند',
                        url: 'https://docs.fandogh.cloud/docs/service-manifest.html#volume-mounts'
                    },
                    password: {
                        title: 'رمز عبور database',
                        text: 'رمز عبوری که برای database خود انتخاب میکنید.',
                        url: '#'
                    }
                },
                sections: {
                    service_config: 'تنظیمات سرویس Mysql',
                    storage_config: 'تعیین محل ذخیره سازی',
                    phpmyadmin_config: 'تنظیمات PhpMyAdmin'
                },
            }
        }, methods: {
            checkBoxSelected(index) {
                this.volumes.forEach(item => {
                    item.selected = false
                })

                this.volumes[index].selected = true
                this.selected_volume = this.volumes[index]
                if (this.selected_volume.label !== 'DedicatedVolume') {
                    this.mysql_manifest.volume_name.value = null
                }
            },
            phpMyAdminSelected() {
                this.phpmyadmin.selected = !this.phpmyadmin.selected
                this.mysql_manifest.phpmyadmin_enabled.value = this.phpmyadmin.selected
            }
        },
        mounted() {

            if (this.manifest_model.parameters.length !== 0) {
                this.manifest_model.parameters.forEach(param => {
                    if (param.name === 'phpmyadmin_enabled') {
                        this.mysql_manifest.phpmyadmin_enabled.value = param.value
                        this.phpmyadmin.selected = param.value
                    } else if (param.name === 'mysql_root_password') {
                        this.mysql_manifest.password.value = param.value
                    } else {
                        this.mysql_manifest.volume_name.value = param.value
                        if (this.mysql_manifest.volume_name.value !== null) {
                            this.volumes.forEach(item => {
                                item.selected = false
                            });

                            this.volumes[1].selected = true
                            this.selected_volume = this.volumes[1]
                        }
                    }
                })
            }

        }, watch: {
            'mysql_manifest.phpmyadmin_enabled': {
                handler: function (value, oldvalue) {
                    this.manifest_model.parameters.forEach((param, index) => {
                        if (param.name === 'phpmyadmin_enabled') {
                            this.manifest_model.parameters.splice(index, 1)
                        }
                    });
                    this.manifest_model.parameters.push(value)
                }, deep: true
            },
            'mysql_manifest.password': {
                handler: function (value, oldvalue) {
                    this.manifest_model.parameters.forEach((param, index) => {
                        if (param.name === 'mysql_root_password') {
                            this.manifest_model.parameters.splice(index, 1)
                        }
                    });
                    if (value.value.toString().length > 0)
                        this.manifest_model.parameters.push(value)
                }, deep: true
            },
            'mysql_manifest.volume_name': {
                handler: function (value, oldvalue) {
                    this.manifest_model.parameters.forEach((param, index) => {
                        if (param.name === 'volume_name') {
                            this.manifest_model.parameters.splice(index, 1)
                        }
                    });
                    if (value.value !== null)
                        this.manifest_model.parameters.push(value)
                }, deep: true
            },
        }
    }
</script>

<style lang="stylus" scoped>

</style>
