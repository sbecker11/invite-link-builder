/* eslint-disable no-console */
<template>
    <v-container style="border:1px solid black;">
        <v-row>
            <v-col cols="6" pr-1>
                <div class="font-weight-thin">Market</div>
                <v-select 
                v-model="market"
                @input="$emit('input', 'locale')"
                :items="markets"
                menu-props="auto"
                label="Select market"
                hide-details
                single-line
                outlined
                dense
                ></v-select>
            </v-col>
            <v-col cols="6" pl-1>
                <div>Language of Recipient</div>
                <v-select 
                id="language-selector"
                v-model="language"
                @input="$emit('input', 'locale')"
                :items="languages"
                menu-props="auto"
                label="Select language"
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
        name: LocalSelector,
        mounted() {
            this.initMarketLanguageTables();
        },
        props: {
            default_locale: String
        },
        data: function() {
            return {
                // these are initialized in initMarketLanguageTables
                market: undefined,
                markets: undefined,
                market_codes: undefined,
                language_codes: undefined,
                // language is re-initialized when market is set
                language: undefined
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
                console.log('initMarketLanguageTables() language_codes.length:', this.language_codes.length);

                this.market_codes = new Map();
                this.market_codes.set('United States', 'US');
                this.market_codes.set('Canada','CA');
                this.market_codes.set('French Polynesia', 'PF');
                this.market_codes.set('Phillipines', 'PH');
                console.log('initMarketLanguageTables() market_codes.length:', this.market_codes.length);

                this.market_languages = new Map();
                this.market_languages.set("United States", ['Spanish', 'Chinese', 'Vietnamese', 'English']);
                this.market_languages.set("Canada", ['Chinese', 'French', 'English']);
                this.market_languages.set("French Polynesia", ['French']);
                this.market_languages.set("Phillipines", ['English']);
                console.log('initMarketLanguageTables() market_languages.length:', this.market_languages.length);

                this.markets = this.market_languages.keys();
                console.log('initMarketLanguageTables() market.length:', this.markets.length);

                // set market and default_lang from locale property
                this.market = this.getMarketFromLocale(this.default_locale);
                console.log('initMarketLanguageTables() market:', this.market);

                // this.languages is recomputed when this.market changes

                let default_lang = this.getLanguageFromLocale(this.default_locale);
                if ( this.languages.indexOf(default_lang) === -1 ) {
                    // if default_lang not found in current lenguages
                    // then set default_lang to first item of languages
                    default_lang = this.languages[0];
                }
                this.language = default_lang;
                console.log('initMarketLanguageTables() language:', this.language);
            },
            getMarketFromLocale(locale) {
                let parts = locale.split('_');
                let market_code = parts[1];
                let find_market = undefined;
                for ( let mkt of this.market_codes.keys() ) {
                    if ( this.market_codes[mkt] === market_code ) {
                        find_market = mkt;
                    }
                }
                return find_market;
            },
            getLanguageFromLocale(locale) {
                let parts = locale.split('_');
                let language_code = parts[0];
                let find_language = undefined;
                for ( let lng of this.language_codes.keys() ) {
                    if ( find_language === 'undefined' ) {
                        if ( this.language_codes[lng] === language_code ) {
                            find_language = lng;
                        }
                    }
                }
                return find_language;
            },
            getMarketCode() {
                // return the market_code of the current market
                let find_market_code = undefined;
                if ( this.market !== undefined ) {
                    find_market_code = this.market_codes[this.market];
                }
                console.log('getMarketCode() market:', this.market);
                console.log('getMarketCode() find_market_code:', find_market_code);
                return find_market_code;
            },
            getLanguageCode() {
                // return the language_code for the current language
                let find_language_code = undefined;
                if ( this.language !== undefined ) {
                    find_language_code = this.languages_codes[this.language];
                }
                console.log('getLanguageCode() language:', this.language);
                console.log('getLanguageCode() find_language_code:', find_language_code);
                return find_language_code;
            }
        },
        computed: {
            locale: function() {
                let current_locale = undefined;
                let language_code = this.getLanguageCode();
                let market_code = this.getMarketCode();
                if ((language_code !== undefined) && (market_code !== undefined)) {
                    current_locale = `${language_code}_${market_code}`;
                }
                return current_locale;
            },
            languages: function() {
                // this function is invoked on any change of:
                // this.market or this.markets
                // returns the current array of languages for the current market
                // this.language SHOULD NOT be altered here.

                // lazy initialize this.markets and this.market
                if ( this.markets === undefined ) {
                    this.initMarketLanguageTables();
                }
                return this.market_languages[this.market];
            },
            language_selector_disabled: function() {
                return this.languages.length == 1;
            }
        }
    }
</script>

<style lang="scss" scoped>

</style>