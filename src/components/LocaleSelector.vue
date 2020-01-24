/* eslint-disable no-console */
<template>
    <v-container id="locale-selector" style="padding:0">
        <v-row>
            <v-col cols="6" id="country-col">
                <div>{{countrySelectorLabel}}</div>
                <v-select 
                id="country-selector"
                v-model="country"
                :items="countrys"
                menu-props="auto"
                :label="countrySelectorPlaceholder"
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
                // eslint-disable-next-line vue/require-valid-default-prop
                default: {}
            },
            countrySelectorLabel: {
                type: String,
                default: 'Market'
            },
            countrySelectorPlaceholder: {
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
                country_codes: this.getCountryCodes(),
                language_codes: this.getLanguageCodes(),
                country_languages: this.getCountryLanguages(),
                countrys: this.getCountrys(),

                locale: this.getDefaultLocaleFromInputValue(this.value),
                country: this.getDefaultCountryFromInputValue(this.value),
                languages: this.getDefaultLanguagesFromInputValue(this.value),
                language: this.getDefaultLanguageFromInputValue(this.value)
            }
        },
        methods: {
            log(...params) {
                console.log(...params);
            },
            getLanguageCodes() {
                if ( this.language_codes === undefined ) {
                    let language_codes = new Map();
                    language_codes.set('Spanish', 'es');
                    language_codes.set('English', 'en');
                    language_codes.set('Chinese', 'zh');
                    language_codes.set('French', 'fr');
                    language_codes.set('Vietnamese', 'vi');
                    let items = Array.from(language_codes.keys());
                    this.log('getLanguageCodes() language_codes.length:', items.length);
                    this.language_codes = language_codes;
                }
                return this.language_codes;
            },
            getCountryCodes() {
                if ( this.country_codes === undefined ) {
                    let country_codes = new Map();
                    country_codes.set('United States', 'US');
                    country_codes.set('Canada','CA');
                    country_codes.set('French Polynesia', 'PF');
                    country_codes.set('Phillipines', 'PH');
                    let items = Array.from(country_codes.keys());
                    this.log('getCountryCodes() country_codes.length:', items.length);
                    this.country_codes = country_codes;
                }
                return this.country_codes;
            },
            getCountryLanguages() {
                if ( this.country_languages === undefined ) {
                    let country_languages = new Map();
                    country_languages.set("United States", ['Spanish', 'Chinese', 'Vietnamese', 'English']);
                    country_languages.set("Canada", ['Chinese', 'French', 'English']);
                    country_languages.set("French Polynesia", ['French']);
                    country_languages.set("Phillipines", ['English']);
                    let items = Array.from(country_languages.keys());
                    this.log('getCountryLanguages() country_languages.length:', items.length);
                    this.country_languages = country_languages;
                }
                return this.country_languages;
            },
            getCountrys() {
                if ( this.countrys === undefined ) {
                    let countrys = Array.from(this.getCountryLanguages().keys());
                    this.log('getCountrys() countrys.length:', countrys.length);
                    this.countrys = countrys;
                }
                return this.countrys;
            },
            getDefaultLocaleFromInputValue(value) {
                if ( this.locale === undefined ) {
                    // validate input value property
                    if ( value === undefined ) {
                        throw new Error('input value is required');
                    }
                    if ( !value.hasOwnProperty('locale') || typeof(value.locale) != 'string' ) {
                        throw new Error('input value missing required "locale" string property');
                    }
                    let locale = value.locale;
                    let localeIsValid = false;
                    if ( locale.length >= 5 ) {
                        let parts = locale.split('_');
                        if ( parts.length == 2 ) {
                            localeIsValid = true;
                        }
                    }
                    if ( !localeIsValid ) {
                        throw new Error('vvalue.locale is invalid');
                    }
                    this.locale = locale;
                }
                return this.locale;
            },
            getDefaultCountryFromInputValue(value) {
                if ( this.country === undefined ) {
                    this.locale = this.getDefaultLocaleFromInputValue(value);
                    this.country = this.getCountryFromLocale(this.locale);
                }
                return this.country;
            },
            getDefaultLanguagesFromInputValue(value) {
                if ( this.languages === undefined ) {
                    this.country = this.getDefaultCountryFromInputValue(value);
                    this.languages = this.getLanguagesForCountry(this.country);
                }
                return this.languages;
            },
            getDefaultLanguageFromInputValue(value) {
                if ( this.language === undefined ) {
                    this.locale = this.getDefaultLocaleFromInputValue(value);
                    this.languages = this.getDefaultLanguagesFromInputValue(value);
                    let defaultLanguage = this.getLanguageFromLocale(this.locale);
                    if ( this.languages.indexOf(defaultLanguage) === -1 ) {
                        this.language = this.getFirstLanguageOfLanguages(this.languages)
                    } else {
                        this.language = defaultLanguage;
                    }
                }
                return this.language;
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
                //this.log("getFirstKeyWithValue() value:", value, " key:", find_key);
                return find_key;
            },
            getValueForKey(map, key) {
                let find_value = map.get(key);
                //this.log("getValueForKey() key:", key, " value:", find_value);
                return find_value;

            },
            getCountryFromLocale(locale) {
                let find_country = undefined;
                if ( locale !== undefined ) {
                    let parts = locale.split('_');
                    let country_code = parts[1];
                    find_country = this.getFirstKeyWithValue(this.country_codes, country_code);
                }
                this.log('getCountryFromLocale() locale:', locale, ' country:', find_country);
                return find_country;
            },
            getLanguageFromLocale(locale) {
                let find_language = undefined;
                if ( locale !== undefined ) {
                    let parts = locale.split('_');
                    let language_code = parts[0];
                    find_language = this.getFirstKeyWithValue(this.language_codes, language_code);
                }
                this.log("getLanguageFromLocale() locale:", locale, " language:", find_language);
                return find_language;
            },
            getCountryCodeFromCountry(country) {
                let find_country_code = undefined;
                if ( country !== undefined ) {
                    find_country_code = this.getValueForKey(this.country_codes, country);
                }
                this.log('getCountryCodeFromCountry() country:', country, ' country_code:', find_country_code);
                return find_country_code;
            },
            getLanguageCodeFromLanguage(language) {
                let find_language_code = undefined;
                if ( language !== undefined ) {
                    find_language_code = this.getValueForKey(this.language_codes, language);
                }
                this.log('getLanguageCodeFromLanguage() language:', language, " language_code:", find_language_code);
                return find_language_code;
            },
            getLanguagesForCountry(country) {
                this.log("getLanguagesForCountry() country:", country);
                let new_languages = undefined;
                if ( this.country_languages !== undefined ) {
                    new_languages = this.getCountryLanguages().get(country);
                }
                // this.log("getLanguagesForCountry() country:", country, " languages:", new_languages);
                return new_languages;
            },
            getFirstLanguageOfLanguages(languages) {
                let first_language = undefined;
                if ( languages !== undefined ) {
                    first_language = languages[0];
                }
                this.log("getFirstLanguageOfLanguages() languages:", languages, " first_language:", first_language);
                return first_language;
            },
            getLocaleFromCountryAndLanguage(country, language) {
                let new_locale = undefined;
                let country_code = this.getCountryCodeFromCountry(country);
                let language_code = this.getLanguageCodeFromLanguage(language);
                if ((language_code !== undefined) && (country_code !== undefined)) {
                    new_locale = `${language_code}_${country_code}`;
                }
                this.log("getLocaleFromCountryAndLanguage() country:", country, " language:", language, " locale:", new_locale);
                return new_locale;
            },
            updateValue() {
                this.log('updateValue()');
                if ( this.country !== undefined && this.language !== undefined && this.locale !== undefined ) {
                    let new_value = {
                        country: this.country,
                        language: this.language,
                        locale: this.locale
                    };
                    if ( this.value !== new_value ) {
                        this.value = new_value;
                        this.log("updateValue() emit value:", this.value);
                        this.$emit('input', this.value);
                    }
                }
            }
        },
        watch: {
            country: function(newCountry, oldCountry) {
                this.log('$$$$ watch.countryProp() newCountry:', newCountry, ' oldCountry:', oldCountry);

                this.country = newCountry;
                this.log('watch.countryProp() this.country:', this.country);

                this.languages = this.getLanguagesForCountry(this.country);
                this.log('watch.countryProp() languages:', this.languages);

                this.language = this.getFirstLanguageOfLanguages(this.languages);
                this.log('watch.countryProp() language:', this.language);

                this.locale = this.getLocaleFromCountryAndLanguage(this.country, this.language);
                this.log('watch.countryProp() locale:', this.locale);

                this.updateValue();
            },
            language: function(newLanguage, oldLanguage) {
                this.log('$$$$ watch.languageProp() newLanguage:', newLanguage, ' oldLanguage:', oldLanguage);

                this.language = newLanguage;
                this.log('watch.languageProp() language:', this.language);

                this.locale = this.getLocaleFromCountryAndLanguage(this.country, this.language);
                this.log('watch.language() locale:', this.locale);

                this.updateValue();
            }
        },
        computed: {
            languageSelectorDisabled: function() {
                this.log("computed.languageSelectorDisabled() this.country:", this.country);
                let disabled = false;
                let num_languages = 2;
                let languages = this.getLanguagesForCountry(this.country);
                if ( languages !== undefined  ) {
                    num_languages = languages.length;
                }
                if ( num_languages <= 1 ) {
                    disabled = true;
                }
                this.log("computed.languageSelectorDisabled() num_languages:",  num_languages, " disabled:", disabled);
                return disabled;
            }
        }
    }
</script>

<style lang="scss" scoped>
#country-col {
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
