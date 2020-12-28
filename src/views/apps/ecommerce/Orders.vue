<template>
	<div class="page-ecommerce-orders flex column">
		
		<div class="page-header card-base header-accent">
			<h1>
				<span>Ecommerce orders</span>
				<button>new order</button>
			</h1>
			<div class="flex justify-space-between">
				<el-breadcrumb separator="/">
					<el-breadcrumb-item :to="{ path: '/' }"><i class="mdi mdi-home-outline"></i></el-breadcrumb-item>
					<el-breadcrumb-item :to="{ path: '/ecommerce' }">Ecommerce</el-breadcrumb-item>
					<el-breadcrumb-item>Orders</el-breadcrumb-item>
				</el-breadcrumb>
				<input placeholder="Search...">
			</div>
		</div>

		<resize-observer @notify="handleResize" /> 

		<div id="table-box" class="bg-white card-shadow--small b-rad-4 box grow" v-loading="resizing">
			<grid 
				id="grid"
				:data="gridProps.data" 
				:columns="gridProps.columns" 
				:bodyHeight="gridProps.bodyHeight" 
				:frozenCount="gridProps.frozenCount" 
				:virtualScrolling="gridProps.virtualScrolling" 
				:minRowHeight="gridProps.minRowHeight" 
				:pagination="gridProps.pagination" 
				:rowHeaders="gridProps.rowHeaders" 
				:header="gridProps.header" 
				:columnOptions="gridProps.columnOptions" 
				:summary="gridProps.summary" 
				v-if="!resizing"
			/>
        </div>
	
	</div>
</template>

<script>
import 'tui-grid/dist/tui-grid.css';
import { Grid } from '@toast-ui/vue-grid';
import Chance from 'chance'
const chance = new Chance()

export default {
	name: 'EcommerceOrders',
	data() {
		const year = new Date().getFullYear()
		const shipping_list = ['UPS', 'Drone', 'FedEx']
		const status_list = ['Complete', 'Pending', 'Returned', 'Paid']
		const gridData = []

		_.times(100, (number) => {
			let price = chance.floating({ min: 1, max: 100, fixed: 2 })
			let qnt = chance.integer({ min: 1, max: 5 })
			let amount = price * qnt

			gridData.push({
				photo: '/static/images/shop/'+chance.integer({ min: 0, max: 19 })+'.jpg',
				product: chance.sentence({ words: 3 }),
				avatar: '/static/images/users/user-'+chance.integer({ min: 0, max: 30 })+'.jpg',
				customer: chance.name(),
				city: chance.city(),
				address: chance.address(),
				email: chance.email(),
				phone: chance.phone(),
				price: price.toFixed(2).toString(),
				qnt: qnt,
				amount: amount.toFixed(2).toString(),
				status: status_list[chance.integer({ min: 0, max: 3 })],
				date: chance.date({string: true, year: year}),
				shipping: shipping_list[chance.integer({ min: 0, max: 2 })],
				id: '#73983'+number
			})
		})
		
		return {
			grid: null,
			gridData,
			resizing: false,
			height: 300,
			gridProps: {
				bodyHeight: null,
				frozenCount: window.innerWidth <= 768 ? 0 : 2,
				virtualScrolling: true,
				minRowHeight: 60,
				pagination: false,
				rowHeaders: ['rowNum'],
				header: {
					height: 50,
					complexColumns: [
						{
							header: 'PRODUCT',
							name: 'mergeColumn1',
							childNames: ['photo', 'product']
						},
						{
							header: 'CUSTOMER',
							name: 'mergeColumn2',
							childNames: ['avatar', 'customer', 'email', 'phone']
						},
						{
							header: 'LOCATION',
							name: 'mergeColumn3',
							childNames: ['city', 'address']
						}
					]
				},
				columnOptions: {
					//minWidth: 100,
					frozenCount: window.innerWidth <= 768 ? 0 : 2,
				},
				columns: [
					{
						header: 'Photo',
						name: 'photo',
						align: 'center',
						width: 60,
						formatter: function(data) {
							var url = data.value.toString();
							return '<img src="' + url + '" width="50" height="50" />';
						}
					},
					{
						header: 'Title',
						name: 'product',
						minWidth: 200,
						sortable: true
					},
					{
						header: 'Status',
						name: 'status',
						minWidth: 150,
						sortable: true,
						formatter: function(data) {
							var value = data.value.toString();
							return '<span class="' + _.kebabCase(value) + '">'+value+'</span>';
						}
					},
					{
						header: 'Avatar',
						name: 'avatar',
						align: 'center',
						width: 60,
						formatter: function(data) {
							var url = data.value.toString();
							return '<img src="' + url + '" width="50" height="50" />';
						}
					},
					{
						header: 'Name',
						name: 'customer',
						minWidth: 200,
						sortable: true
					},
					{
						header: 'Email',
						name: 'email',
						minWidth: 200,
						sortable: true,
					},
					{
						header: 'Phone',
						name: 'phone',
						minWidth: 200,
						sortable: true,
					},
					{
						header: 'City',
						name: 'city',
						minWidth: 200,
						sortable: true
					},
					{
						header: 'Address',
						name: 'address',
						minWidth: 200,
						sortable: true
					},
					{
						header: 'Quantity',
						name: 'qnt',
						minWidth: 100,
						sortable: true
					},
					{
						header: 'Amount',
						name: 'amount',
						minWidth: 100,
						sortable: true,
						formatter: function(data) { return '<strong>$ '+_.replace(data.value.toString(), '.', ',')+'</strong>'; }
					},
					{
						header: 'Date',
						name: 'date',
						minWidth: 100,
						sortable: true
					},
					{
						header: 'Shipping',
						name: 'shipping',
						minWidth: 100,
						sortable: true
					},
					{
						header: 'Order',
						name: 'id',
						minWidth: 100,
						sortable: true
					}
				],
				summary: {
					height: 30,
					position: 'bottom', // or 'top'
					columnContent: {
						amount: {
							template: function(valueMap) {
								return '<small>AVG: <strong>$ ' + valueMap.avg.toFixed(2) + '</strong></small>';
							}
						},
						qnt: {
							template: function(valueMap) {
								return '<small>AVG: <strong>' + valueMap.avg.toFixed(0) + '</strong></small>';
							}
						}
					}
				},
				data: gridData
			}
		}
	},
	methods: {
		handleResize: _.throttle(function (e) {
			console.log('resize', this.resizing)
			if(!this.resizing) {
				this.resizing = true
				setTimeout(()=>{this.resizing = false;},1000)
				setTimeout(()=>{this.initGrid()},1500)
			}
		}, 1000),
		gotoNew() {
			this.$router.push({name:'ecommerce-new-product'})
		},
		initGrid() {
			this.gridProps.bodyHeight = document.getElementById('table-box').clientHeight - 71
			this.gridProps.frozenCount = window.innerWidth <= 768 ? 0 : 2
			this.gridProps.columnOptions.frozenCount = window.innerWidth <= 768 ? 0 : 2
		}
	},
	mounted() {
		this.initGrid()		
	},
	components: {
		Grid
	}
}
</script>

<style lang="scss">
@import '../../../assets/scss/_variables';

.page-ecommerce-orders {
	.page-header {
		margin-bottom: 8px;

		h1 {
			button {
				float: right;
				padding: 5px 10px;
				background: rgba(255, 255, 255, .2);
				text-transform: uppercase;
				border: none;
				outline: none;
				cursor: pointer;
				color: white;
				opacity: .8;
				font-family: inherit;

				i {
					position: relative;
					top: 3px;
				}

				&:hover {
					opacity: 1;
				}
			}
		}

		input {
			background: rgba(255, 255, 255, .2);
			border: none;
			outline: none;
			height: 20px;
			margin-top: 5px;
			padding: 4px 8px;
			min-width: 200px;
			font-family: inherit;
			color: white;
			font-size: 16px;
		}
	}

	#table-box {
		overflow: hidden;
	}

	#grid {
		.tui-grid-cell[data-column-name="photo"] ,
		.tui-grid-cell[data-column-name="avatar"] {
			.tui-grid-cell-content {
				padding: 0; 
			}
		}

		.tui-grid-cell[data-column-name="photo"] ,
		.tui-grid-cell[data-column-name="product"] {
			background: white;
		}

		.tui-grid-border-line-top {
			background-color: transparent;
			border-color: white;
		}
		.tui-grid-cell-head {
			border-top-color: transparent;
		}
	
		.tui-grid-cell-head[data-column-name="mergeColumn1"] ,
		.tui-grid-cell-head[data-column-name="mergeColumn2"] ,
		.tui-grid-cell-head[data-column-name="mergeColumn3"] ,
		.tui-grid-cell-head[data-column-name="mergeColumn4"] ,
		.tui-grid-cell-head[data-column-name="mergeColumn5"] ,
		.tui-grid-cell-head[data-column-name="mergeColumn6"] {
			background: #f4f4f4;
			color: #8e8e8e;
		}

		.tui-grid-cell-content {
			.pending {
				color: #f7ba2a;
				font-weight: bold;
			}
			.returned {
				color: #ff005c;
				font-weight: bold;
			}
			.complete {
				color: #1ac367;
				font-weight: bold;
			}
			.paid {
				color: #2d6dd3;
				font-weight: bold;
			}
		}
	}
}


@media (max-width: 900px) {
	.page-ecommerce-orders {

		.page-header {
			.el-breadcrumb {
				display: none;
			}
		}
	}
}


@media (max-width: 480px) {
	.page-ecommerce-orders {

		.page-header {
			h1 {
				display: none;
			}
		}
	}
}
</style>


