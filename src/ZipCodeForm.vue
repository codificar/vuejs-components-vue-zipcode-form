<template>
	<ValidationObserver
		ref="zipCodeAddressForm"
		tag="form"
		@submit.stop.prevent="sendForm()"
	>
		<vue-element-loading
			:active="loadZipCode"
			spinner="spinner"
			color="#6666FF"
		/>
		<div class="m-2 row" v-if="showAddressName">
			<div class="col">
				<label> {{ trans("common_address.location_name") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.location_name')"
				>
					<input
						v-model="addressForm.location_name"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.location_name')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>
		</div>

		<div class="m-2 row">
			<div class="col">
				<label> {{ trans("common_address.zip_code") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					v-mask="['#####-###']"
					:rules="{ required: true, regex: /[0-9]{5}-[\d]{3}/ }"
					:name="trans('common_address.zip_code')"
				>
					<input
						v-model="addressForm.zip_code"
						@blur="getZipCodeInfo"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.zip_code')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>

			<div class="col">
				<label> {{ trans("common_address.street") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.street')"
				>
					<input
						v-model="addressForm.street"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.street')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>
		</div>
		<div class="m-2 row">
			<div class="col">
				<label> {{ trans("common_address.city") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.city')"
				>
					<input
						v-model="addressForm.city"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.city')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>

			<div class="col">
				<label> {{ trans("common_address.district") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.district')"
				>
					<input
						v-model="addressForm.district"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.district')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>
		</div>

		<div class="m-2 row">
			<div class="col">
				<label> {{ trans("common_address.state") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.state')"
				>
					<input
						v-model="addressForm.state"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.state')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>

			<div class="col">
				<label> {{ trans("common_address.country") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.country')"
				>
					<input
						v-model="addressForm.country"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.country')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>
		</div>

		<div class="m-2 row">
			<div class="col">
				<label> {{ trans("common_address.number") }}*</label>
				<ValidationProvider
					v-slot="{ errors }"
					rules="required"
					:name="trans('common_address.number')"
				>
					<input
						v-model="addressForm.number"
						type="text"
						class="form-control"
						:placeholder="trans('common_address.number')"
					/>
					<div v-if="!!errors[0]" style="color: red">
						{{ errors[0] }}
					</div>
				</ValidationProvider>
			</div>

			<div class="col">
				<label> {{ trans("common_address.complement") }}</label>
				<input
					v-model="addressForm.complement"
					type="text"
					class="form-control"
					:placeholder="trans('common_address.complement')"
				/>
			</div>
		</div>

		<div v-if="showFormButton" class="float-right">
			<button type="submit" class="btn btn-success">
				<i class="mdi mdi-plus"></i> {{ trans("common_address.add_new") }}
			</button>
		</div>
	</ValidationObserver>
</template>

<script>
import { debounce } from "lodash";
import axios from "axios";
import { ValidationObserver, ValidationProvider } from "vee-validate";
import VueElementLoading from "vue-element-loading";
import veeValidate from "./plugins/vee-validate";
veeValidate.configValidate();
export default {
	components: {
		ValidationObserver,
		ValidationProvider,
		VueElementLoading,
	},
	name: "ZipCodeForm",
	props: {
		currentAddress: {
			type: Object,
			default: () => ({}),
		},
		addressesList: {
			type: Array,
			default: () => [],
		},
		showAddressName: {
			type: Boolean,
			default: () => true,
		},
		showFormButton: {
			type: Boolean,
			default: () => true,
		},

		autocompleteParams: {
			type: Object,
			required: true,
		},
		autocompleteUrl: {
			type: String,
			required: true,
		},
		geocodeUrl: {
			type: String,
			required: true,
		},
		zipCodeUrl: {
			type: String,
			required: true,
		},
	},

	watch: {
		addressForm: {
			handler: function (newVal) {
				this.$emit("input", newVal);
			},
			deep: true,
		},
	},

	data() {
		return {
			addressForm: {
				zip_code: "",
				street: "",
				city: "",
				country: "",
				district: "",
				state: "",

				latitude: "",
				longitude: "",

				location_name: "",
				number: "",

				complement: "",

				full_address: "",
			},
			loadZipCode: false,
		};
	},

	methods: {
		trans(key) {
			return _.get(window.lang, key, key);
		},
		async callAutocompleteApi(searchString) {
			const { data: response } = await axios.get(this.autocompleteUrl, {
				params: { ...this.autocompleteParams, place: searchString },
			});

			if (response.success && response.data.length > 0) {
				let addressData = response.data[0];
				if (addressData.place_id != null)
					addressData = await this.callGeocodeApi(addressData.address);

				this.addressForm.latitude = addressData.latitude;
				this.addressForm.longitude = addressData.longitude;
			} else {
				this.$toasted.show(this.trans("common_address.zip_code_not_found"), {
					theme: "bubble",
					type: "warning",
					position: "bottom-center",
					duration: 5000,
				});
			}
		},
		async callGeocodeApi(address) {
			const { data: response } = await axios.get(this.geocodeUrl, {
				params: { ...this.autocompleteParams, address },
			});
			if (response.success) return response.data;
			return false;
		},
		handleZipCodeInput: debounce(async function () {
			await this.getZipCodeInfo();
		}, 400),
		async getZipCodeInfo() {
			if (this.addressForm.zip_code.length > 6) {
				this.loadZipCode = true;
				try {
					const response = await axios.post(this.zipCodeUrl, {
						zipcode: this.addressForm.zip_code,
					});
					this.loadZipCode = false;
					if (response.status === 200 && response.data.success) {
						const data = response.data;
						this.addressForm.street = data.street;
						this.addressForm.city = data.city;
						this.addressForm.district = data.district;
						this.addressForm.state = data.state;
						this.addressForm.latitude = data.latitude;
						this.addressForm.longitude = data.longitude;
					}
				} catch (error) {
					this.loadZipCode = false;
					this.addressForm.latitude = "";
					this.addressForm.longitude = "";
					this.$toasted.show(this.trans("common_address.zip_code_not_found"), {
						theme: "bubble",
						type: "warning",
						position: "bottom-center",
						duration: 5000,
					});
				}
			}
		},
		resetForm() {
			this.addressForm = {
				zip_code: "",
				street: "",
				city: "",
				country: "",
				district: "",
				state: "",

				latitude: "",
				longitude: "",

				location_name: "",
				number: "",

				complement: "",

				full_address: "",
			};
		},
		async validateForm() {
			const validator = await this.$refs.zipCodeAddressForm.validate();
			if (validator && this.addressForm.latitude && this.addressForm.longitude)
				return true;
			return false;
		},
		async sendForm() {
			const validator = await this.validateForm();
			if (!validator) {
				return;
			}

			this.addressForm.full_address = `${this.addressForm.street} ${this.addressForm.number}, ${this.addressForm.district} - ${this.addressForm.state}, ${this.addressForm.country}`;

			if (
				this.addressForm.latitude === "" ||
				this.addressForm.longitude === ""
			) {
				this.loadZipCode = true;
				await this.callAutocompleteApi(this.addressForm.full_address);
			}

			this.loadZipCode = false;

			if (this.addressForm.latitude && this.addressForm.longitude) {
				this.$emit("on-send-form", this.addressForm);
				this.resetForm();
			} else {
				this.$toasted.show(this.trans("common_address.zip_code_not_found"), {
					theme: "bubble",
					type: "warning",
					position: "bottom-center",
					duration: 5000,
				});
			}
		},
	},
};
</script>
