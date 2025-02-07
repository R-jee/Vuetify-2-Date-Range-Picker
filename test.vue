<template>
    <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
            transition="scale-transition" offset-y min-width="auto">
        <template v-slot:activator="{ on, attrs }">
            <v-text-field
                single-line
                ref="input"
                hide-details="auto"
                v-model="formattedDate"
                :label="selectedPreset || $t('ALL Time')"
                :prepend-inner-icon="icon"
                :placeholder="inputPlaceholder"
                :autofocus="autofocus"
                :autocomplete="autocomplete"
                :append-icon="postIcon || null"
                :reverse="$vuetify.rtl"
                :loading="loading"
                :rules="validationRules"
                :required="required"
                validate-on-blur
                v-bind="attrs"
                v-on="on"
                readonly
                clearable
                @click:clear="clearDate"
            ></v-text-field>
        </template>
        <v-card class="pa-2">
            <v-row dense>
                <v-col dense class="border-r-2" outlined>
                    <v-list-item dense v-for="(preset, key) in presets" :key="key" @click="setPreset(key, preset)">
                        <v-list-item-title>{{ key }}</v-list-item-title>
                    </v-list-item>
                    <v-list-item dense @click="setCustomPreset">
                        <v-list-item-title>Custom</v-list-item-title>
                    </v-list-item>
                </v-col>
                <v-col dense>
                    <v-sheet class="d-flex align-center justify-center pr-4">
                        <v-date-picker
                            dense
                            v-model="dateRange[0]"
                            @update:modelValue="updateDate"
                            no-title
                            scrollable
                            class="picker-width mr-2 picker-no-border"
                            flat
                            tile
                            elevation="0"
                        ></v-date-picker>
                        <v-date-picker
                            dense
                            v-model="dateRange[1]"
                            :min="dateRange[0]"
                            @update:modelValue="updateDate"
                            no-title
                            scrollable
                            class="picker-width picker-no-border"
                            flat
                            tile
                            elevation="0"
                        ></v-date-picker>
                    </v-sheet>
                </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-card-actions>
                <v-text class="mr-4">{{ formattedDate || 'No date selected' }}</v-text>
                <v-spacer></v-spacer>
                <v-btn small color="primary" @click="applyDate">Apply</v-btn>
                <v-btn small color="secondary" @click="clearDate">Clear</v-btn>
            </v-card-actions>
        </v-card>
    </v-menu>
</template>

<script>
export default {
    name: "JetDateRangePicker",
    props: {
        value: {
            type: Array,
            default: () => ["", ""]
        },
        autofocus: Boolean,
        autocomplete: {
            type: String,
            default: "off"
        },
        icon: {
            type: String,
            default: "mdi-calendar"
        },
        postIcon: {
            type: String,
            default: ""
        },
        loading: Boolean,
        required: Boolean,
        placeholder: {
            type: String,
            default: ""
        }
    },
    data: () => ({
        menu: false,
        dateRange: ["", ""],
        selectedPreset: "ALL Time",
        presets: {
            "Today": [new Date().toISOString().substr(0, 10), new Date().toISOString().substr(0, 10)],
            "Yesterday": [
                new Date(new Date().setDate(new Date().getDate() - 1)).toISOString().substr(0, 10),
                new Date(new Date().setDate(new Date().getDate() - 1)).toISOString().substr(0, 10)
            ],
            "Last 7 Days": [
                new Date(new Date().setDate(new Date().getDate() - 6)).toISOString().substr(0, 10),
                new Date().toISOString().substr(0, 10)
            ],
            "Last 30 Days": [
                new Date(new Date().setDate(new Date().getDate() - 29)).toISOString().substr(0, 10),
                new Date().toISOString().substr(0, 10)
            ],
            "This Month": [
                new Date(new Date().getFullYear(), new Date().getMonth(), 1).toISOString().substr(0, 10),
                new Date().toISOString().substr(0, 10)
            ],
            "Last Month": [
                new Date(new Date().getFullYear(), new Date().getMonth() - 1, 1).toISOString().substr(0, 10),
                new Date(new Date().getFullYear(), new Date().getMonth(), 0).toISOString().substr(0, 10)
            ]
        }
    }),
    computed: {
        formattedDate() {
            return this.dateRange[0] && this.dateRange[1] ? `${this.dateRange[0]} - ${this.dateRange[1]}` : "";
        },
        inputPlaceholder() {
            return this.placeholder || this.$t('ALL Time');
        },
        validationRules() {
            return [];
        }
    },
    watch: {
        value: {
            immediate: true,
            handler(newVal) {
                this.dateRange = newVal;
            }
        }
    },
    methods: {
        updateDate() {
            this.$emit('change', {
                dateRange: [this.dateRange[0] ?? "", this.dateRange[1] ?? ""],
                date_filter: this.computePersetTag(this.selectedPreset),
                preset: this.computePersetTag(this.selectedPreset),
                start_date: this.dateRange[0] ?? "",
                end_date: this.dateRange[1] ?? ""
            });
        },
        setPreset(name, range) {
            this.selectedPreset = name;
            this.dateRange = range;
            this.updateDate();
        },
        setCustomPreset() {
            this.selectedPreset = "Custom";
            this.dateRange = ["", ""];
        },
        clearDate() {
            this.selectedPreset = "ALL Time";
            this.dateRange = ["", ""];
            this.$emit('change', {
                dateRange: ["", ""],
                date_filter: this.computePersetTag(this.selectedPreset),
                preset: this.computePersetTag(this.selectedPreset),
                start_date: this.dateRange[0] ?? "",
                end_date: this.dateRange[1] ?? ""
            });
        },
        computePersetTag(selectedPresetTag) {
            let pTag = 'all';
            if (selectedPresetTag === "ALL Time") {
                pTag = 'all';
            } else if (selectedPresetTag === "Today") {
                pTag = 'today';
            } else if (selectedPresetTag === "Yesterday") {
                pTag = 'yesterday';
            } else if (selectedPresetTag === "Last 7 Days") {
                pTag = 'last_7';
            } else if (selectedPresetTag === "Last 30 Days") {
                pTag = 'last_30';
            } else if (selectedPresetTag === "This Month") {
                pTag = 'this_month';
            } else if (selectedPresetTag === "Last Month") {
                pTag = 'last_month';
            } else if (selectedPresetTag === "Custom") {
                pTag = 'custom';
            }
            return pTag;
        },
        applyDate() {
            this.menu = false;
            this.updateDate();
        }
    }
};
</script>

<style scoped>
.v-list {
    max-height: 200px;
    overflow-y: auto;
}

.picker-width {
    max-width: 50%;
}

.picker-no-border {
    border: none !important;
    box-shadow: none !important;
    background: transparent !important;
}
</style>




/*

  <jet-date-range-picker @change="handleClick" />

  handleClick(value) {
      console.log(value);
  },

*/
