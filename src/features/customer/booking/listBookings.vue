<template>
	<div v-if="authenticated=='customer'"class="container" style="background:#fff">
		<div class="row list-table">
			<div v-show="error" class="alert alert-warning alert-dismissible mx-auto" role="alert">
				<strong>{{msg}}</strong>
			</div>
			<b-table responsive hover stacked="sm" :items="bookings" :fields="bookingFields" @row-clicked="toggleDetails" :sort-compare="customSort">

				<template slot="bookingFrom" slot-scope="row">
					{{convertFormat(row.value)}}
				</template>

				<template slot="row-details" slot-scope="row">
					<b-card>
						<b-form @submit.prevent="updateTable(row.item, row.toggleDetails)" @reset="row.toggleDetails">
						</b-form>
					</b-card>
				</template>

			</b-table>
		</div>
	</div>
</template>

<script>
	export default {
		name: "tables",

		components: {
		},

		data () {
			this.getBookings();
			return {
				error: false,
				msg: '',
				authenticated: this.$store.state.authenticated,
				bookings: [],
				bookingFields: [
					{key: 'bookingFrom', label: 'Date', sortable: true},
					{key: 'restaurant.name', label: 'Restaurant', sortable: true},
					{key: 'noOfPersons', label: 'Persons', sortable: true},
					{key: 'bookingStatus', label: 'Status', sortable: true},
				]
			}
		},

		methods: {
			toggleDetails(a) {
				a._showDetails = !a._showDetails;
			},

			customSort(prev, next, col) {
				if (col == 'bookingFrom') {
					return ((new Date(prev.bookingFrom)).getTime() - (new Date(next.bookingFrom)).getTime());
				}

				if (col == 'restaurant.name') {
					return prev.restaurant.name.localeCompare(next.restaurant.name)
				}
			},

			convertFormat(date) {
				return ((new Date(date)).toLocaleString());
			},

			getBookings() {
				console.log('Getting bookings');
				this.$http.get(
					process.env.VUE_APP_API_ROUTE + 'booking/',
					{
						headers: {
							Authorization: 'Bearer ' + JSON.parse(this.$store.state.user).token
						}
					}
				).then((response) => {
					console.log(response);
					if (response.ok && response.body.bookings) {
						// Add _showDetails key to toggle details on table
						// Row color depending upon booking status
						this.bookings = response.body.bookings.map(x => {
							x._showDetails = false ;
							x._rowVariant = (x.bookingStatus == 'confirmed')
								? 'success'
								: (x.bookingStatus == 'canceled')
									? 'active'
									: 'warning';
							return x;
						});
						console.log('Bookings: ', (this.bookings));
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
			}
		}
	}
</script>

<style>
.list-table {
	margin-top: 20px;
}
.fa-trash {
	color: #eaa;
}
.btn-status {
font-size: 10px;
padding: 3px;
margin: 1px;
}
</style>
