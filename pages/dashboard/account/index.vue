<template>
    <div class="row">

        <div class="col-lg-11 col-md-10 col-sm-8 col-xs-12 container-fluid"
             style="box-sizing: content-box; max-width: 1600px; overflow-x: scroll; overflow-y: hidden">

            <p class="title_header">حساب کاربری</p>

            <div class="box row">
                <div class="col-lg-2 col-md-2 col-sm-12 col-xs-12; margin: 0;" style="height: 100%;width: 100%">
                    <div style="border: solid 4px #24d5d8; border-radius: 100px; width: 90px; height: 90px;margin-left: auto; margin-right: auto">
                    </div>

                    <div class="row">
                        <button @click="accountEdit" class="container-fluid left curve-button"
                                style="margin: 24px auto;">ویرایش اطلاعات
                        </button>
                    </div>
                </div>

                <div class="col-lg-5 col-md-4 col-sm-12 col-xs-12 row" style="display: block; margin: 0;">

                    <div style="min-height: 3em;">
                        <p class="profile-entity-title">نام و نام‌خانوادگی:</p>
                        <p class="profile-entity-value">{{fullName}}</p>
                    </div>
                    <div style="min-height: 3em">
                        <p class="profile-entity-title">شماره ملی:</p>
                        <p class="profile-entity-value" style="font-family: iran-sans; font-size: 0.85em">
                            {{account.national_id}}</p>
                    </div>
                    <div style="min-height: 3em">
                        <p class="profile-entity-title">تاریخ عضویت:</p>
                        <p class="profile-entity-value"></p>
                    </div>
                    <div style="min-height: 3em; display: block;">
                        <p class="profile-entity-title" style="display: inline-block">نام فضانام:</p>
                        <v-select
                                style="display: inline-block"
                                :options="namespaces"
                                :clearable="false"
                                :searchable="false"
                                dir="rtl"
                                label="name"
                                v-model="activatedNamespace"
                                placeholder="نام فضانام را انتخاب کنید">
                        </v-select>
                        <img @click="$router.push('/dashboard/account/namespaces')"
                             src="../../../assets/svg/circular-add.svg" alt="add-namespace" class="create-namespace">
                    </div>

                </div>

                <div class="col-lg-5 col-md-6 col-sm-12 col-xs-12 row" style="display: block; margin: 0;">

                    <div style="min-height: 3em;">
                        <p class="profile-entity-title">نام کاربری:</p>
                        <p class="profile-entity-value">{{username}}</p>
                    </div>

                    <div style="min-height: 3em; overflow: hidden; white-space: nowrap;">
                        <p class="profile-entity-title">آدرس ایمیل:</p>
                        <p class="profile-entity-value">{{email}}</p>
                    </div>


                    <div style="min-height: 3em">
                        <p class="profile-entity-title">نوع کاربری:</p>
                        <p class="profile-entity-value" v-if="activeNamespace.quota">
                            {{(activeNamespace.quota.memory_limit/1024 >=1 ? 'حرفه‌ای' : 'رایگان')}}</p>
                    </div>


                    <div style="min-height: 3em">
                        <p class="profile-entity-title">دریافت خبرنامه:</p>
                        <p class="profile-entity-value">
                            {{account.newsletter_subscriber ? 'دریافت میکنم' : 'تایید نکرده‌ام'}}</p>
                    </div>

                </div>
            </div>

            <div class="box-row row">
                <div @click="sectionClicked('ProfilePlan')"
                     :class="[(activeSectionName === 'ProfilePlan' ? 'enabled' : 'disabled')]">
                    <p :style="{borderLeft: '1px solid #1757FF'}">پلن من</p>
                </div>

                <!--<div v-if="verifyUserAccess({ADMIN:'ADMIN'})" @click="sectionClicked('ProfileWallet')"-->
                <!--:class="[(activeSectionName === 'ProfileWallet' ? 'enabled' : 'disabled')]">-->
                <!--<p :style="{borderLeft: '1px solid #1757FF'}">کیف پول</p>-->
                <!--</div>-->

                <div v-if="verifyUserAccess({ADMIN:'ADMIN'})" @click="sectionClicked('ProfileTransactions')"
                     :class="[(activeSectionName === 'ProfileTransactions' ? 'enabled' : 'disabled')]">
                    <p :style="{borderLeft: '1px solid #1757FF'}">تراکنش‌های مالی</p>
                </div>

                <div @click="sectionClicked('TeamManagement')"
                     :class="[(activeSectionName === 'TeamManagement' ? 'enabled' : 'disabled')]">
                    <p :style="{borderLeft: '1px solid #1757FF'}">مدیریت تیم</p>
                </div>

                <!--<div @click="sectionClicked('ProfileMessages')"-->
                <!--:class="[(activeSectionName === 'ProfileMessages' ? 'enabled' : 'disabled')]">-->
                <!--<p>پیام‌های من</p>-->
                <!--</div>-->
            </div>

            <keep-alive>
                <component v-bind:is="activeSectionName"></component>
            </keep-alive>

        </div>
    </div>
</template>

<script>
    import FInput from "~/components/elements/input";
    import FButton from "~/components/elements/button";
    import ProfilePlan from "~/components/Dashboard/profile/profile-plan";
    import ProfileWallet from "~/components/Dashboard/profile/profile-wallet";
    import ProfileTransactions from "~/components/Dashboard/profile/profile-transaction";
    import TeamManagement from "~/components/Dashboard/profile/team-management";
    import File from "~/components/elements/file";
    import {formData} from "~/utils/formData";
    import ProgressBar from "~/components/Dashboard/progress-bar";
    import ErrorReporter from "~/utils/ErrorReporter";
    import FormValidator from "~/utils/formValidator";
    import FLabelDisable from "~/components/elements/label/label-disable";
    import FCheckbox from "~/components/elements/checkbox";
    import {removeValue, setValue, getValue} from "../../../utils/cookie";
    import Moment from 'moment-jalaali';
    import RoleAccessHandler from "../../../utils/RoleAccessHandler";

    export default {
        layout: "dashboard",
        name: "account",
        components: {
            FInput,
            FButton,
            File,
            ProgressBar,
            FLabelDisable,
            FCheckbox,
            ProfilePlan,
            ProfileWallet,
            ProfileTransactions,
            TeamManagement,
            Moment
        },
        data() {
            return {
                loading: false,
                loadingProgress: false,
                activeSectionName: 'ProfilePlan',
                namespaces: [],
                activeNamespace: {},
                dateJoined: '',
            };
        },
        computed: {
            activatedNamespace: {
                get: function () {
                    return this.activeNamespace;
                }, set: function (namespace) {
                    if (this.$route.query.ns !== namespace.name) {
                        sessionStorage.setItem('user_role', namespace.user_role);
                        sessionStorage.setItem('namespace', namespace.name);
                        this.$store.commit('SET_DATA', {data: false, id: 'loading'});
                        window.location.replace(this.$route.path + '?ns=' + namespace.name);
                    }
                }
            },
            openSidebar() {
                return this.$store.state.sideMunu
            },
            account() {
                return this.$store.state.account;
            },
            email() {
                return getValue("email");
            },
            progress() {
                return this.$store.state.progress;
            },
            username() {
                return getValue("username");
            },
            namespace() {
                return getValue('namespace')
            }, fullName() {
                return this.account.first_name + ' ' + this.account.last_name;
            }
        },

        mounted() {
            this.$ga.event({
                eventCategory: "account",
                eventAction: "show information"
            });
        },
        created() {
            this.getData();
            if (this.$route.query.ns)
                this.fetchUserNamespaces();
        },
        methods: {
            verifyUserAccess(permitted_roles) {
                return RoleAccessHandler(permitted_roles)
            },
            async fetchUserNamespaces() {
                this.namespaces.length = 0;
                try {
                    this.namespaces = await this.$store.dispatch('requestUserNamespaces');
                    for (let i = 0; i < this.namespaces.length; i++) {
                        if (this.namespaces[i].name === this.$route.query.ns) {
                            this.activeNamespace = this.namespaces[i];
                            sessionStorage.setItem('user_role', this.namespaces[i].user_role);
                            sessionStorage.setItem('namespace', this.namespaces[i].name);
                            break;
                        }
                    }
                    this.$store.commit('SET_DATA', {data: false, id: 'loading'})
                } catch (e) {
                    this.$store.commit("SET_DATA", {data: false, id: "loading"});
                    if (e.status === 401) {
                        this.$router.push("/user/login");
                    } else {
                        ErrorReporter(e, this.$data, true).forEach(error => {
                            this.$notify({
                                title: error,
                                time: 4000,
                                type: "error"
                            });
                        });

                    }
                }
            },
            async getData() {
                try {
                    let res = await this.$store.dispatch("getAccount", {
                        username: getValue("username")
                    });

                    if (res.newsletter_subscriber) {
                        let elm = document.querySelector("#newsletter_subscriber");
                        elm.click();
                    }
                    this.dateJoined = Moment(res.date_joined).format('jYYYY/jMM/jDD');
                    this.fetchUserNamespaces();
                } catch (e) {
                    this.$store.commit("SET_DATA", {data: false, id: "loading"});
                    if (e.status === 401) {
                        this.$router.push("/user/login");
                    } else {
                        ErrorReporter(e, this.$data, true).forEach(error => {
                            this.$notify({
                                title: error,
                                time: 4000,
                                type: "error"
                            });
                        });
                    }
                }
            },
            accountEdit() {
                this.$ga.event({
                    eventCategory: "account",
                    eventAction: "click btn edit information"
                });
                this.$router.push("/dashboard/account/edit");
            }, sectionClicked(sectionName) {
                this.activeSectionName = sectionName;
            }
        }
    };
</script>


<style lang="stylus" scoped>

    @import '../../../assets/css/variables.styl'

    .box-checkbox
        display flex
        @media only screen and (max-width: 768px)
            flex-direction column
            > div
                margin-bottom 15px

    .title_header
        font-family iran-yekan
        font-style normal
        font-weight bold
        font-size 1.2em
        font-stretch normal
        line-height 1.75
        color #7c7c7c
        letter-spacing normal

    .profile-entity-title
        font-family yekan-bold
        font-size .9em
        min-width 120px
        font-style normal
        display inline-block
        font-stretch normal
        margin 0
        line-height 1.75
        letter-spacing normal
        color #111111
        @media only screen and (max-width: 1200px)
            min-width 75px

    .profile-entity-value
        font-family iran-yekan
        font-size .9em
        display inline-block
        text-align right
        line-height 1.75
        text-overflow ellipsis
        margin 0
        margin-right 10px
        letter-spacing normal
        color #000000


    .box
        padding 16px
        box-sizing content-box
        border-radius 3px
        box-shadow 0 2px 6px 0 rgba(0, 0, 0, 0.07)
        background-color #ffffff

    .box-row
        height fit-content
        border-radius 3px
        box-shadow 0 2px 6px 0 rgba(0, 0, 0, 0.07)
        background-color #ffffff
        margin-top 5px
        display flex
        margin-bottom 5px
        white-space nowrap
        overflow-x scroll
        overflow-y hidden
        -ms-overflow-style none
        scrollbar-width none
        @media only screen and (max-width 992px)
            display block
            border-radius 0

        div.disabled
            display inline-flex
            padding 0
            margin-left -5px
            cursor pointer

            p
                font-style normal
                font-stretch normal
                min-width 200px
                line-height 24px
                margin-top 8px
                text-align center
                font-family iran-yekan
                font-size .9em
                outline none
                margin-bottom 8px
                letter-spacing normal
                color #000000


        div.enabled
            display inline-flex
            padding 0
            margin-right -1px
            background-color $colorBlueArea
            cursor pointer

            p
                font-style normal
                font-stretch normal
                min-width 200px
                line-height 24px
                text-align center
                margin-top 8px
                font-family yekan-bold
                font-size .9em
                outline none
                margin-bottom 8px
                letter-spacing normal
                color #ffffff

    .box-row::-webkit-scrollbar
        display none


    .plan-container
        border-radius 3px
        box-shadow 0 2px 6px 0 rgba(0, 0, 0, 0.07)
        background-color #ffffff

    .curve-button
        border-radius 17.5px
        padding 10px 36px
        background-color #7ed321
        color #ffffff
        font-family iran-yekan
        font-size 14px
        border none
        outline none
        cursor pointer

    .create-namespace
        display inline-block
        width 32px
        height 32px
        margin 0
        cursor pointer
        margin-right 12px
        vertical-align middle
        @media only screen and (max-width: 1100px)
            width 27px
            height 27px
</style>

<style lang="stylus">

    .v-select .vs__dropdown-toggle
        margin-bottom 8px !important
        min-height 25px !important
        border none !important
        border-radius 10px !important
        background-color #f5f5f5 !important
        min-width 200px !important
        max-width 250px !important
        direction rtl !important
        @media only screen and (max-width: 1100px)
            min-width 175px !important

    .v-select .vs__selected-tag
        font-family iran-yekan !important
        color: #000000 !important
        font-size .9em !important
        padding-right 5px !important
        font-weight normal !important
        font-style normal !important

    span.vs__selected
        padding-right 8px !important

    div.vs__actions
        margin-left 3px !important
        margin-top 3px !important


</style>
