<template>
	<v-card width="400" class="mx-auto mt-5" background-color="#F6F7F7">
		<v-card-text>
            <h1 class="text-center">Create Invitation</h1>
            <LocaleSelector
                v-model="locale"
                defaultMarket="United States"
                marketSelectorLabel="Market"
                marketSelectorPlaceholder="Select Market"
                languageSelectorLabel="Language of Recipient"
                languageSelectorPlaceholder="Select Language"
            />
            <v-container style="border:1px solid black;">
                <v-row>
                    <v-col cols="12">
                        <div>Link Type</div>
                        <v-btn-toggle v-model="link_type" tile>
                        <v-btn value="cust" width="105">Customer</v-btn>
                        <v-btn value="memb" width="105">Member</v-btn>
                        <v-btn value="brnd" width="105">Brand<br>Affiliate</v-btn>
                        </v-btn-toggle>
                    </v-col>
                </v-row>
            </v-container>
            <v-container>
                <v-row align="center" justify="center">
                    <v-col cols=12>
                        <p class="text-center" v-html="link_type_description"/>
                    </v-col>
                </v-row>
            </v-container>
            <v-container>
                <v-row>
                    <v-divider></v-divider>
                </v-row>
            </v-container>
            <v-container>
                <v-row align="center" justify="center">
                    <v-col cols=12>
                        <h1 class="text-center">Share Invitation</h1>
                    </v-col>
                </v-row>
            </v-container>
            <v-container>
                <v-row>
                  <v-col>
                    <v-card outlined flat>
                        <v-card-text>
                            <div class="text-center">
                                Brand Affiliate invite from<br>
                                {{sponsor_name}} - {{sponsor_id}}<br>
                                {{locale.market}} &middot; {{locale.language}}
                            </div>
                            <v-card outlined flat class="mt-2 mb-0" style="padding:1em;">{{current_url}}</v-card>
                            <input type="hidden" id="hidden-url-element" :value="current_url">

                        </v-card-text>
                        <v-card-actions>
                            <v-btn block color="#7123d9" dark 
                                @click="copyUrlToClipboard">
                                Copy Link
                            </v-btn>
                            <v-snackbar 
                                v-model="snackbar_visible">
                                {{snackbar_text}}
                            </v-snackbar>
                        </v-card-actions>
                    </v-card>
                  </v-col>
                </v-row>
            </v-container>
        </v-card-text>
	</v-card>
</template>

<script>
import LocaleSelector from '@/components/LocaleSelector.vue'

export default {
    name: 'invite',
    components: {
        LocaleSelector
    },
	data() {
        return {
            sponsor_id: 'US00340640',
            sponsor_name: 'Danielle Anderson',
            locale: {
                locale: 'en_US',
                market: '',
                language: ''
            },
            link_type: 'brnd',
            snackbar_visible: false,
            snackbar_text: undefined
		}
    },
    methods: {
        copyUrlToClipboard () {
            // get the text value of "hidden-url-element"
            // copy it to the copy buffer
            // display the results in a snackbar
            // hide the "hidden-url-element"

            let hiddenUrlElement = document.querySelector('#hidden-url-element')
            hiddenUrlElement.setAttribute('type', 'text')    // 不是 hidden 才能複製
            hiddenUrlElement.select()

            try {
                var successful = document.execCommand('copy');
                this.snackbar_text = successful ? "Link Copied to Clipboard!" : "link copy failed";
            } catch (err) {
                this.snackbar_text = "link copy had errors";
            }
            this.snackbar_visible = true;

            /* unselect the range */
            hiddenUrlElement.setAttribute('type', 'hidden');
            window.getSelection().removeAllRanges()
        }
    },
    computed: {

        link_type_description: function() {
            let desc = '';
            switch(this.link_type) {
                case 'cust':
                    desc += '<span class="font-weight-black">';
                    desc += 'Customers';
                    desc += '</span>';
                    desc += ' are wonderful';
                    break;
                case 'memb':
                    desc += '<span class="font-weight-black">';
                    desc += 'Members';
                    desc += '</span>';
                    desc += ' are fantastic';
                    break;
                case 'brnd':
                    desc += '<span class="font-weight-black">';
                    desc += 'Brand Affiliates';
                    desc += '</span>';
                    desc += ' are monumental';
                    break;
                default:
                    desc = '';
                    break;
            }
            return desc;
        },
        current_url: function() {
            if ((this.link_type !== undefined) &&
                (this.locale !== undefined)) { 
                return `https://${this.link_type}/${this.sponsor_id}/${this.locale.locale}`;
            } else {
                return '';
            }
        }
    }
}
</script>

<style></style>
