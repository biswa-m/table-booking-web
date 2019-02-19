<template>
		<div class="booking-restauranteur">
			<h3 class="text-left">
				<small>{{selectedList}} of </small>
				<b>{{$store.state.restaurant.name}}</b>
			</h3>
			<div v-show="error" class="alert alert-warning alert-dismissible mx-auto" role="alert">
				<strong>{{msg}}</strong>
			</div>
			<hr>
			<div class="row mr-1">
				<div class="ml-auto">
					<button class="btn btn-outline-info btn-sm"
									@click="showFilter=!showFilter">
						Filter Bookings
						<i class="fa fa-caret-down" aria-hidden="true"></i>
					</button>
				</div>
			</div>
			<transition name="slide">
				<div v-show="showFilter" class="row">
					<div class="pt-1 pb-2 w-100 mx-3">
						<b-card header="Filter" title="">
							<div class="row">
								<div class="col-sm-3 text-left text-sm-right">
									<b>Status:</b>
								</div>
								<div class="col-sm-9">
									<b-form-checkbox-group v-model="params.bookingStatus"
										:options="['pending', 'confirmed', 'canceled']">
									</b-form-checkbox-group>
								</div>
							</div>
							<div class="row">
								<b-button variant="primary btn-sm ml-auto mr-3 mt-3"
													@click="getBookings(params); showFilter=false;">Done
								</b-button>
							</div>
						</b-card>
					</div>
				</div>
			</transition>
			<b-row class="mt-1">
				<b-col md=9>
					<div>
						<b-pagination
							:limit="8"
							:total-rows="count"
							:per-page="limit"
							v-model="currentPage"
							class="my-0"
							@change="paginate"
						/>
					</div>
				</b-col>
				<b-col md=3 class="flex">
					<span class="my-auto ml-0 mr-2">Per page</span>
					<b-form-select :options="pageOptions" class="per-page" v-model="limit" />
				</b-col>
			</b-row>
			<booking-list :restaurant="$store.state.restaurant"
											:bookings="bookings"
											:disabledDates="disabledDates">
			</booking-list>
			<b-row class="mt-1">
				<b-col md=9>
					<div>
						<b-pagination
							:limit="8"
							:total-rows="count"
							:per-page="limit"
							v-model="currentPage"
							class="my-0"
							@change="paginate"
						/>
					</div>
				</b-col>
				<b-col md=3 class="flex">
					<span class="my-auto ml-0 mr-2">Per page</span>
					<b-form-select :options="pageOptions" class="per-page" v-model="limit" />
				</b-col>
			</b-row>
		</div>
	</div>
</template>

<script>
	import bookingList from "@/features/restauranteur/booking/bookingList.vue"

	export default {
		name: "select-booking-list",

		props: {
			selectedList: null,
			disabledDates: {}
		},

		components: {
			'booking-list': bookingList 
		},

		data () {
			return {
				error: false,
				msg: '',

				bookings: [],
				showFilter: false,
				params: {},

				pageOptions: [5, 10, 15, 20, 25, 30],
				count: 0,
				limit: 20,
				skip: 0,
				currentPage: 0
			}
		},

		computed: {
			restaurantId() {
				return this.$store.state.restaurant.id;
			}
		},

		watch: {
			restaurantId() {
				this.getBookings(this.params);
			},

			selectedList(newValue, oldValue) {
				// Whent different list is selected update booking list
				this.updateBookingList(newValue);
			},

			limit(value) {
				this.skip = 0;
				this.currentPage = 1;
				this.getBookings(this.params);
			}
		},

		mounted() {
			this.updateBookingList(this.selectedList);
		},

		methods: {
			getBookings(params) {
				params.skip = this.skip;
				params.limit = this.limit;
				console.log('Getting bookings: ', JSON.stringify(params));
				this.$http.get(
					process.env.VUE_APP_API_ROUTE + 'restaurant/bookings/' + this.$store.state.restaurant.id,
					{
						params,
						headers: {
							Authorization: 'Bearer ' + JSON.parse(this.$store.state.user).token
						}
					}
				).then((response) => {
					console.log(response);
					if (response.ok && response.body.bookings) {
						// Add customerDetails key to view details of customer on each booking 
						// Add _showDetails key to toggle details on each booking 
						// Add rowVariant key to change style of each booking depending on booking status
						this.bookings = response.body.bookings.map(x => {
							x.customerDetails = {};
							x._showDetails = false ;
							x._rowVariant = (x.bookingStatus == 'confirmed')
								? 'success'
								: (x.bookingStatus == 'canceled')
									? 'active'
									: 'warning';
							return x;
						});

						// Total no of booking
						this.count = response.body.count
					} else {
						this.error = true;
						this.msg = (response.body.errors && response.body.errors.message)
							? response.body.errors.message
							: "Could not get the bookings";
					}
				}).catch((e) => {
					this.error = true;
					this.msg = (e.body && e.body.errors && e.body.errors.message)
						? e.body.errors.message
						: "Could not get the bookings";
					console.log(JSON.stringify(e));
				});
			},
			updateBookingList(item) {
				if (item == 'Upcoming bookings') {
					this.params.before = null;
					this.params.after = (new Date()).getTime();
					this.getBookings(this.params);
				}
				if (item == 'Past bookings') {
					this.params.before = (new Date()).getTime();
					this.params.after = null;
					this.getBookings(this.params);
				}
				if (item == 'All bookings') {
					this.params.before = null;
					this.params.after = null;
					this.getBookings(this.params);
				}
			},

			paginate(page) {
				this.skip = (page - 1) * this.limit;
				this.getBookings(this.params);
				console.log('Page number: ', page, 'skip: ', this.skip);
			}
		}
	}
</script>

<style>
.fa-trash {
	color: #eaa;
}
.btn-status {
font-size: 10px;
padding: 3px;
margin: 1px;
}
.slide-enter-active {
  transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-leave-active {
  transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-enter, .slide-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}
.flex-container {
	display: flex;
	justify-content: left;
}
.booking-restauranteur .form-control[readonly] {
	background-color: white;
}
.booking-restauranteur .form-control:disabled {
	background-color: #e9ecef;
}
.per-page {
	max-width: 60px;
}
</style>