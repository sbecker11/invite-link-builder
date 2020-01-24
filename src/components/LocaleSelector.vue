/* eslint-disable no-console */
<template>
    <v-container id="locale-selector" style="padding:0">
        <v-row>
            <v-col cols="6" id="market-col">
                <div>{{marketSelectorLabel}}</div>
                <v-select 
                id="market-selector"
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
            <v-col cols="6" id="language-col">
                <div>{{languageSelectorLabel}}</div>
                <v-select 
                id="language-selector"
                v-model="language"
                :items="languages"
                menu-props="auto"
                :disabled="languageSelectorDisabled ? true : false"
                :append-icon="languageSelectorDisabled ? '' : undefined"
                :label="languageSelectorPlaceholder"
                hide-details
                single-line
                outlined
                dense
                ></v-select>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
    export default {
        props: {
            // reactive property
            // parent uses <LocaleSelector v-model="locale"/> and locale in data 
            value: {
                type: Object,
                default: function() {}
            },
            defaultMarket: {
                type: String,
                default: 'United States'
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
                // these are lazily initialized
                market_codes: this.getMarketCodes(),
                language_codes: this.getLanguageCodes(),
                market_languages: this.getMarketLanguages(),
                markets: this.getMarkets(),

                market: this.defaultMarket,
                languages: this.getDefaultLanguages(),
                language: this.getDefaultLanguage(),
                locale: this.getDefaultLocale()
            }
        },
        methods: {
            getLanguageCodes() {
                if ( this.language_codes === undefined ) {
                    let language_codes = new Map();
                    language_codes.set('Spanish', 'es');
                    language_codes.set('English', 'en');
                    language_codes.set('Chinese', 'zh');
                    language_codes.set('French', 'fr');
                    language_codes.set('Vietnamese', 'vi');
                    let items = Array.from(language_codes.keys());
                    console.log('getLanguageCodes() language_codes.length:', items.length);
                    this.language_codes = language_codes;
                }
                return this.language_codes;
            },
            getMarketCodes() {
                if ( this.market_codes === undefined ) {
                    let market_codes = new Map();
                    market_codes.set('United States', 'US');
                    market_codes.set('Canada','CA');
                    market_codes.set('French Polynesia', 'PF');
                    market_codes.set('Phillipines', 'PH');
                    let items = Array.from(market_codes.keys());
                    console.log('getMarketCodes() market_codes.length:', items.length);
                    this.market_codes = market_codes;
                }
                return this.market_codes;
            },
            getMarketLanguages() {
                if ( this.market_languages === undefined ) {
                    let market_languages = new Map();
                    market_languages.set("United States", ['Spanish', 'Chinese', 'Vietnamese', 'English']);
                    market_languages.set("Canada", ['Chinese', 'French', 'English']);
                    market_languages.set("French Polynesia", ['French']);
                    market_languages.set("Phillipines", ['English']);
                    let items = Array.from(market_languages.keys());
                    console.log('getMarketLanguages() market_languages.length:', items.length);
                    this.market_languages = market_languages;
                }
                return this.market_languages;
            },
            getMarkets() {
                if ( this.markets === undefined ) {
                    let markets = Array.from(this.getMarketLanguages().keys());
                    console.log('getMarkets() markets.length:', markets.length);
                    this.markets = markets;
                }
                return this.markets;
            },
            getDefaultLanguages() {
                return this.getLanguagesForMarket(this.defaultMarket);
            },
            getDefaultLanguage() {
                let defaultLanguages = this.getDefaultLanguages();
                console.log("getDefaultLanguage() defaultLanguages:", defaultLanguages);
                let defaultLanguage = this.getFirstLanguageForLanguages(defaultLanguages);
                console.log("getDefaultLanguage() defaultLanguage:", defaultLanguage);
                return defaultLanguage;
            },
            getDefaultLocale() {
                let defaultLanguage = this.getDefaultLanguage();
                let defaultLocale = this.getLocaleFromMarketAndLanguage(this.defaultMarket, defaultLanguage);
                return defaultLocale;
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
            getMarketCodeFromMarket(market) {
                let find_market_code = undefined;
                if ( market !== undefined ) {
                    find_market_code = this.getValueForKey(this.market_codes, market);
                }
                console.log('getMarketCodeFromMarket() market:', market, ' market_code:', find_market_code);
                return find_market_code;
            },
            getLanguageCodeFromLanguage(language) {
                let find_language_code = undefined;
                if ( language !== undefined ) {
                    find_language_code = this.getValueForKey(this.language_codes, language);
                }
                console.log('getLanguageCodeFromLanguage() language:', language, " language_code:", find_language_code);
                return find_language_code;
            },
            getLanguagesForMarket(market) {
                console.log("getLanguagesForMarket() market:", market);
                let new_languages = undefined;
                if ( this.market_languages !== undefined ) {
                    new_languages = this.getMarketLanguages().get(market);
                }
                // console.log("getLanguagesForMarket() market:", market, " languages:", new_languages);
                return new_languages;
            },
            getFirstLanguageForLanguages(languages) {
                let first_language = undefined;
                if ( languages !== undefined ) {
                    first_language = languages[0];
                }
                console.log("getFirstLanguageForLanguages() languages:", languages, " first_language:", first_language);
                return first_language;
            },
            getLocaleFromMarketAndLanguage(market, language) {
                let new_locale = undefined;
                let market_code = this.getMarketCodeFromMarket(market);
                let language_code = this.getLanguageCodeFromLanguage(language);
                if ((language_code !== undefined) && (market_code !== undefined)) {
                    new_locale = `${language_code}_${market_code}`;
                }
                console.log("getLocaleFromMarketAndLanguage() market:", market, " language:", language, " locale:", new_locale);
                return new_locale;
            },
            updateValue() {
                console.log('updateValue()');
                if ( this.market !== undefined && this.language !== undefined && this.locale !== undefined ) {
                    let new_value = {
                        market: this.market,
                        language: this.language,
                        locale: this.locale
                    };
                    if ( this.value !== new_value ) {
                        this.value = new_value;
                        console.log("updateValue() emit value:", this.value);
                        this.$emit('input', this.value);
                    }
                }
            }
        },
        watch: {
            market: function(newMarket, oldMarket) {
                console.log('$$$$ watch.marketProp() newMarket:', newMarket, ' oldMarket:', oldMarket);

                this.market = newMarket;
                console.log('watch.marketProp() this.market:', this.market);

                this.languages = this.getLanguagesForMarket(this.market);
                console.log('watch.marketProp() languages:', this.languages);

                this.language = this.getFirstLanguageForLanguages(this.languages);
                console.log('watch.marketProp() language:', this.language);

                this.locale = this.getLocaleFromMarketAndLanguage(this.market, this.language);
                console.log('watch.marketProp() locale:', this.locale);

                this.updateValue();
            },
            language: function(newLanguage, oldLanguage) {
                console.log('$$$$ watch.languageProp() newLanguage:', newLanguage, ' oldLanguage:', oldLanguage);

                this.language = newLanguage;
                console.log('watch.languageProp() this..language');

                this.locale = this.getLocaleFromMarketAndLanguage(this.market, this.language);
                console.log('watch.language() locale:', this.locale);

                this.updateValue();
            }
        },
        computed: {
            languageSelectorDisabled: function() {
                console.log("computed.languageSelectorDisabled() this.market:", this.market);
                let disabled = false;
                let num_languages = 2;
                let languages = this.getLanguagesForMarket(this.market);
                if ( languages !== undefined  ) {
                    num_languages = languages.length;
                }
                if ( num_languages <= 1 ) {
                    disabled = true;
                }
                console.log("computed.languageSelectorDisabled() num_languages:",  num_languages, " disabled:", disabled);
                return disabled;
            }
        }
    }
</script>

<style lang="scss" scoped>
#market-col {
    padding-right: 3px;
    font-weight: 100;
    font-family: Arial;
}
#language-col {
    padding-left: 3px;
    font-weight: 100;
    font-family: Arial;
}
</style>
