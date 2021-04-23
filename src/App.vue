<template>
	<div id="app">
		<div id="panel">
			<div class="fld-search">
				Search by ticker:
				<input type="text" v-on:input="onInputSearch" />
			</div>
			<div class="fld-fiat-switch">
				Fiat switch:
				<select v-model="displayFiatFor">
					<option value="ALL">Все</option>
					<option value="USD">USD</option>
					<option value="RUR">RUR</option>
					<option value="EUR">EUR</option>
				</select>
			</div>
		</div>
		<table id="main-table">
			<thead>
				<tr v-bind:class="[sortDirectionClass]">
					<th
						v-on:click="onApplyTickerSort"
						v-bind:class="{ sort: isTickerSort }"
					>
						Ticker
					</th>
					<th
						v-if="isDisplayedUsd"
						v-on:click="onApplyFiatSortUsd"
						v-bind:class="{ sort: isUsdSort }"
					>
						USD
					</th>
					<th
						v-if="isDisplayedRur"
						v-on:click="onApplyFiatSortRur"
						v-bind:class="{ sort: isRurSort }"
					>
						RUR
					</th>
					<th
						v-if="isDisplayedEur"
						v-on:click="onApplyFiatSortEur"
						v-bind:class="{ sort: isEurSort }"
					>
						EUR
					</th>
				</tr>
			</thead>
			<tbody>
				<tr v-for="item in coinList" :key="item.ticker">
					<td>{{ item.ticker }}</td>
					<td v-if="isDisplayedUsd">{{ item.prices.USD }}</td>
					<td v-if="isDisplayedRur">{{ item.prices.RUR }}</td>
					<td v-if="isDisplayedEur">{{ item.prices.EUR }}</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<script>
import _ from "lodash";
import axios from "axios";

const SORT_DIRECTION = Object.freeze({
	ASC: "asc",
	DESC: "desc",
	NONE: null,
});

const PATH_TABLE = Object.freeze({
	TICKER: "ticker",
	USD: "prices.USD",
	RUR: "prices.RUR",
	EUR: "prices.EUR",
});

export default {
	name: "App",
	data: function () {
		return {
			priceData: {},
			sortInfo: {},
			filter: "",
			displayFiatFor: "ALL",
		};
	},
	created: function () {
		this.$__resetSortInfo();
	},
	mounted: function () {
		axios({
			method: "GET",
			url:
				"https://min-api.cryptocompare.com/data/pricemulti?fsyms=btc,eth,doge,xrp,trx,xlm,bch,bsv,neo,ada,awc,go,xtz,qtum,smart,ltc,xmr,vet,vtho,dent,eos,dcn,nexo,tpay,zrx,usdc,usdt,bnb,tnt,rex,rep,rcn,kin&tsyms=USD,EUR,RUR",
		}).then((response) => {
			this.priceData = response.data;
		});
	},
	computed: {
		coinList: function () {
			return _.chain(this.priceData)
				.map((prices, ticker) => {
					return {
						ticker,
						prices: _.defaults({}, prices),
					};
				})
				.filter((val) => {
					return _.startsWith(val.ticker, this.filter);
				})
				.orderBy([this.sortInfo.path], [this.sortInfo.direction])
				.value();
		},
		isTickerSort: function () {
			return this.$__isSortByPath(PATH_TABLE.TICKER);
		},
		isUsdSort: function () {
			return this.$__isSortByPath(PATH_TABLE.USD);
		},
		isRurSort: function () {
			return this.$__isSortByPath(PATH_TABLE.RUR);
		},
		isEurSort: function () {
			return this.$__isSortByPath(PATH_TABLE.EUR);
		},
		isDisplayedUsd: function () {
			return this.$__isFiatSwitchedDisplayFor("USD");
		},
		isDisplayedRur: function () {
			return this.$__isFiatSwitchedDisplayFor("RUR");
		},
		isDisplayedEur: function () {
			return this.$__isFiatSwitchedDisplayFor("EUR");
		},
		sortDirectionClass: function () {
			return this.sortInfo.direction;
		},
	},
	methods: {
		onApplyTickerSort: function () {
			this.$__applySort(PATH_TABLE.TICKER);
		},
		onApplyFiatSortUsd: function () {
			this.$__applySort(PATH_TABLE.USD);
		},
		onApplyFiatSortRur: function () {
			this.$__applySort(PATH_TABLE.RUR);
		},
		onApplyFiatSortEur: function () {
			this.$__applySort(PATH_TABLE.EUR);
		},
		onInputSearch: function ({ target }) {
			this.filter = _.toUpper(target.value);
		},
		//
		$__applySort: function (path) {
			if (this.sortInfo.path === path) {
				this.sortInfo.direction = this.$__nextSortDirection(
					this.sortInfo.direction
				);
				if (this.sortInfo.direction === SORT_DIRECTION.NONE) {
					this.$__resetSortInfo();
				}
			} else {
				this.sortInfo = {
					path,
					direction: this.$__nextSortDirection(null),
				};
			}
		},
		$__nextSortDirection: function (direction) {
			switch (direction) {
				case SORT_DIRECTION.ASC:
					return SORT_DIRECTION.DESC;
				case SORT_DIRECTION.DESC:
					return SORT_DIRECTION.NONE;
				default:
					return SORT_DIRECTION.ASC;
			}
		},
		$__resetSortInfo: function () {
			this.sortInfo = {
				path: null,
				direction: null,
			};
		},
		$__isSortByPath: function (path) {
			return this.sortInfo.path === path;
		},
		$__isFiatSwitchedDisplayFor: function (code) {
			return _.indexOf(["ALL", code], this.displayFiatFor) !== -1;
		},
	},
};
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		margin: auto;
		margin-top: 60px;
		width: 40%;
	}
	#app .space {
		height: 50px;
	}

	#main-table {
		font-family: Arial, Helvetica, sans-serif;
		border-collapse: collapse;
		width: 100%;
	}

	#main-table td, #main-table th {
		border: 1px solid #ddd;
		padding: 8px;
	}

	#main-table tr:nth-child(even){
		background-color: #f2f2f2;
	}

	#main-table tr:hover {
		background-color: #ddd;
	}

	#main-table th {
		padding-top: 12px;
		padding-bottom: 12px;
		text-align: center;
		background-color: #4CAF50;
		color: white;
		cursor: pointer;
	}

	#main-table tr td { 
		text-align: right; 
	}

	#main-table tr td:first-child { 
		text-align: center;
	}

	#main-table thead tr th:not(.sort)::after {
		content: "\00a0\00a0";
	}

	#main-table tr.asc th.sort:after {
		content: "\02191";
	}

	#main-table tr.desc th.sort:after {
		content: "\02193";
	}

	#panel {
		display: flex;
		justify-content: start;
		align-items: center;
		width: 100%;
		height: 70px;
	}

	#panel > div {
		margin-right: 20px;
	}

	#panel input, #panel select {
		font-size: 16px;
		outline: none;
	}
</style>
