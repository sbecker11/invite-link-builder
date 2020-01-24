/* eslint-disable no-console */
<template>
    <v-container style="border:1px solid black;">
        <v-row>
            <v-col cols="6" pr-1>
                <div class="font-weight-thin">{{marketSelectorLabel}}</div>
                <v-select 
                v-model="market"
                :items="markets"
                menu-props="auto"
                :label="marketSelectorPlaceholder"
                hide-details
                single-line
                outlined
                dense
                ></v-select>
            </v-col>
            <v-col cols="6" pl-1>
                <div>{{languageSelectorLabel}}</div>
                <v-select 
                id="language-selector"
                v-model="language"
                :items="languages"
                menu-props="auto"
                :label="languageSelectorPlaceholder"
                hide-details
                single-line
                outlined
                dense
                :disabled="language_selector_disabled ? true : false"
                :append-icon="language_selector_disabled ? '' : undefined"
                ></v-select>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
    export default {
        mounted() {
            this.initMarketLanguageTables();
        },
        props: {
            // reactive property
            // parent uses <LocaleSelector v-model="locale"/> and locale in data 
            value: {
                type: Object,
                default: {}
            },
            marketSelectorLabel: {
                type: String,
                default: 'Market'
            },
            marketSelectorPlaceholder: {
                type: String,
                default: 'Select Market'
            },
            languageSelectorLabel: {
                type: String,
                default: 'Language of Recipient'
            },
            languageSelectorPlaceholder: {
                type: String,
                default: 'Select Language'
            }
        },
        data: function() {
            return {
                // these are initialized in initMarketLanguageTables
                markets: undefined,
                market_codes: undefined,
                language_codes: undefined,

                // this.market is initialized in initMarketLanguageTables
                market: "",

                // this.language is re-initialized when this.market changes
                language: ""
            }
        },
        methods: {
            initMarketLanguageTables() {

                this.language_codes = new Map();
                this.language_codes.set('Spanish', 'es');
                this.language_codes.set('English', 'en');
                this.language_codes.set('Chinese', 'zh');
                this.language_codes.set('French', 'fr');
                this.language_codes.set('Vietnamese', 'vi');
                let items = Array.from(this.language_codes.keys());
                console.log('initMarketLanguageTables() language_codes.length:', items.length);

                this.market_codes = new Map();
                this.market_codes.set('United States', 'US');
                this.market_codes.set('Canada','CA');
                this.market_codes.set('French Polynesia', 'PF');
                this.market_codes.set('Phillipines', 'PH');
                items = Array.from(this.market_codes.keys());
                console.log('initMarketLanguageTables() market_codes.length:', items.length);

                this.market_languages = new Map();
                this.market_languages.set("United States", ['Spanish', 'Chinese', 'Vietnamese', 'English']);
                this.market_languages.set("Canada", ['Chinese', 'French', 'English']);
                this.market_languages.set("French Polynesia", ['French']);
                this.market_languages.set("Phillipines", ['English']);
                items = Array.from(this.market_languages.keys());
                console.log('initMarketLanguageTables() market_languages.length:', items.length);

                this.markets = Array.from(this.market_languages.keys());
                console.log('initMarketLanguageTables() markets.length:', this.markets.length);

                // set market and default_lang from locale property
                this.market = this.getMarketFromLocale(this.value.locale);
                console.log('initMarketLanguageTables() market:', this.market);

                // this.languages is recomputed when this.market changes

                // adjust this.language and this.value.local if locale-defined 
                // language is not found in this.languages for this.market.
                let default_lang = this.getLanguageFromLocale(this.value.locale);
                if ( this.languages !== undefined && this.languages.indexOf(default_lang) === -1 ) {
                    // if default_lang not found in current lenguages
                    // then set default_lang to first item of languages
                    default_lang = this.languages[0];
                }
                this.language = default_lang;
                console.log('initMarketLanguageTables() language:', this.language);
            },
            getFirstKeyWithValue(map, value) {
                let find_key = undefined;
                if ( map !== undefined ) {
                    for ( let entry of map.entries() ) {
                        if ( find_key === undefined ) {
                            if ( entry[1] === value ) {
                                find_key = entry[0];
                            }
                        }
                    }
                }
                //console.log("getFirstKeyWithValue() value:", value, " key:", find_key);
                return find_key;
            },
            getValueForKey(map, key) {
                let find_value = map.get(key);
                //console.log("getValueForKey() key:", key, " value:", find_value);
                return find_value;

            },
            getMarketFromLocale(locale) {
                let find_market = undefined;
                if ( locale !== undefined ) {
                    let parts = locale.split('_');
                    let market_code = parts[1];
                    find_market = this.getFirstKeyWithValue(this.market_codes, market_code);
                }
                console.log('getMarketFromLocale() locale:', locale, ' market:', find_market);
                return find_market;
            },
            getLanguageFromLocale(locale) {
                let find_language = undefined;
                if ( locale !== undefined ) {
                    let parts = locale.split('_');
                    let language_code = parts[0];
                    find_language = this.getFirstKeyWithValue(this.language_codes, language_code);
                }
                console.log("getLanguageFromLocale() locale:", locale, " language:", find_language);
                return find_language;
            },
            getMarketCode(market) {
                // return the market_code of the current market
                let find_market_code = undefined;
                if ( market !== undefined ) {
                    find_market_code = this.getValueForKey(this.market_codes, market);
                }
                console.log('getMarketCode() market:', market, ' market_code:', find_market_code);
                return find_market_code;
            },
            getLanguageCode(language) {
                // return the language_code for the current language
                let find_language_code = undefined;
                if ( language !== undefined ) {
                    find_language_code = this.getValueForKey(this.language_codes, language);
                }
                console.log('getLanguageCode() language:', language, " language_code:", find_language_code);
                return find_language_code;
            },
            updateLocale() {
                let current_locale = undefined;
                let language_code = this.getLanguageCode(this.language);
                let market_code = this.getMarketCode(this.market);
                if ((language_code !== undefined) && (market_code !== undefined)) {
                    current_locale = `${language_code}_${market_code}`;
                }
                if ( (current_locale !== undefined) && (current_locale !== this.value.locale)) {
                    this.value = {
                        locale: current_locale,
                        market: this.market,
                        language: this.language
                    };
                    this.$emit('input', this.value);
                }
            }
        },
        watch: {
            value: function(new_locale) {
                let new_market = this.getMarketFromLocale(new_locale.locale);
                if ( new_market !== undefined && this.market !== new_market ) {
                    this.market = new_market;
                }
                let new_language = this.getLanguageFromLocale(new_locale.locale);
                if ( new_language !== undefined && this.language !== new_language ) {
                    this.language = new_language;
                }
            },
            market: function() {
                this.updateLocale();
            },
            language: function() {
                this.updateLocale();
            }
        },
        computed: {
            languages: function() {
                // this function is invoked on any change of:
                // this.market or this.markets
                // returns the current array of languages for the current market
                // this.language SHOULD NOT be altered here.

                // lazy initialize this.markets and this.market
                if ( this.markets === undefined ) {
                    this.initMarketLanguageTables();
                }
                let new_languages = this.market_languages.get(this.market);
                console.log("market:", this.market, " languages:", new_languages);
                return new_languages;
            },
            language_selector_disabled: function() {
                return this.languages !== undefined && this.languages.length == 1;
            }
        }
    }
</script>

<style lang="scss" scoped>

</style>
