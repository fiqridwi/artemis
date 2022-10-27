<template>
	<div>
		<div>
			<h1 style="text-align: center">{{ viewName }}</h1>
			<button @click="handleGo()">Go to Jakarta</button> ||
			<button @click="goToHeatMapLayer">Heatmap</button> ||
			<button @click="activateProvinceLayer()">Province</button>
			<button @click="disableProvinceLayer()">Disable Province</button>||
			<button @click="activateMarker()">Marker</button>
			<button @click="disableMarker()">Disable Marker</button> ||
			<button @click="activateGeopoint()">Geopoint</button>
			<button @click="disableGeopoint()">Disable Geopoint</button> ||
			<button @click="activateFOLayer()">FO</button>
			<button @click="disableFOLayer()">Disable FO</button> ||
			<button @click="activateBTSLayer()">BTS</button>
		</div>

		<div>
			<div className="map-wrap">
				<div ref="mapContainer" class="map" />
			</div>
		</div>
	</div>
</template>
<script setup>
import { ref, onMounted } from "vue";
import maplibregl from "maplibre-gl";
import jsonData from "../../src/assets/lamudi_crawl_results.json";

const viewName = ref("Artemis");
let map = ref(null);
const mapContainer = ref(null);
let data = ref(null);
let markers = ref([]);
let token = ref(
	"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImEyZmE2MDQzLTMwOTYtNGZkNy05NGU2LTAwMjA3NDRiZTRiMyIsInJvbGUiOiI4NzczZmVhNy04ZTYyLTRhZTUtODQ3ZS0xMjQwNDIzMzFmZTciLCJhcHBfYWNjZXNzIjp0cnVlLCJhZG1pbl9hY2Nlc3MiOnRydWUsImlhdCI6MTY2Njc4MDM2OSwiZXhwIjoxNjY2NzgxMjY5LCJpc3MiOiJkaXJlY3R1cyJ9.IZjRxJEi_m-u06BQ77C6ysgyTi3ZctDSebXVAJYT_Tw"
);

onMounted(() => {
	map = new maplibregl.Map({
		container: mapContainer.value,
		style: "https://api.maptiler.com/maps/basic-v2/style.json?key=aJ1aWBQyxkpsVjqWZPpk",
		center: [106.845599, -6.21462],
		zoom: 10,
	});

	// Map Controls
	map.addControl(
		new maplibregl.NavigationControl({
			visualizePitch: true,
			showZoom: true,
			showCompass: true,
		})
	);
});

const handleGo = () => {
	map.flyTo({
		center: [106.845599, -6.21462],
		zoom: 8,
	});
};

//heatmap map libre
const goToHeatMapLayer = () => {
	map.flyTo({
		center: [-120, 50],
		zoom: 3,
	});
	// Add a geojson point source.
	// Heatmap layers also work with a vector tile source.
	map.addSource("earthquakes", {
		type: "geojson",
		data: "https://maplibre.org/maplibre-gl-js-docs/assets/earthquakes.geojson",
	});

	map.addLayer(
		{
			id: "earthquakes-heat",
			type: "heatmap",
			source: "earthquakes",
			maxzoom: 9,
			paint: {
				// Increase the heatmap weight based on frequency and property magnitude
				"heatmap-weight": [
					"interpolate",
					["linear"],
					["get", "mag"],
					0,
					0,
					6,
					1,
				],
				// Increase the heatmap color weight weight by zoom level
				// heatmap-intensity is a multiplier on top of heatmap-weight
				"heatmap-intensity": [
					"interpolate",
					["linear"],
					["zoom"],
					0,
					1,
					9,
					3,
				],
				// Color ramp for heatmap.  Domain is 0 (low) to 1 (high).
				// Begin color ramp at 0-stop with a 0-transparancy color
				// to create a blur-like effect.
				"heatmap-color": [
					"interpolate",
					["linear"],
					["heatmap-density"],
					0,
					"rgba(33,102,172,0)",
					0.2,
					"rgb(103,169,207)",
					0.4,
					"rgb(209,229,240)",
					0.6,
					"rgb(253,219,199)",
					0.8,
					"rgb(239,138,98)",
					1,
					"rgb(178,24,43)",
				],
				// Adjust the heatmap radius by zoom level
				"heatmap-radius": [
					"interpolate",
					["linear"],
					["zoom"],
					0,
					2,
					9,
					20,
				],
				// Transition from heatmap to circle layer by zoom level
				"heatmap-opacity": [
					"interpolate",
					["linear"],
					["zoom"],
					7,
					1,
					9,
					0,
				],
			},
		},
		"waterway"
	);

	map.addLayer(
		{
			id: "earthquakes-point",
			type: "circle",
			source: "earthquakes",
			minzoom: 7,
			paint: {
				// Size circle radius by earthquake magnitude and zoom level
				"circle-radius": [
					"interpolate",
					["linear"],
					["zoom"],
					7,
					["interpolate", ["linear"], ["get", "mag"], 1, 1, 6, 4],
					16,
					["interpolate", ["linear"], ["get", "mag"], 1, 5, 6, 50],
				],
				// Color circle by earthquake magnitude
				"circle-color": [
					"interpolate",
					["linear"],
					["get", "mag"],
					1,
					"rgba(33,102,172,0)",
					2,
					"rgb(103,169,207)",
					3,
					"rgb(209,229,240)",
					4,
					"rgb(253,219,199)",
					5,
					"rgb(239,138,98)",
					6,
					"rgb(178,24,43)",
				],
				"circle-stroke-color": "white",
				"circle-stroke-width": 1,
				// Transition from heatmap to circle layer by zoom level
				"circle-opacity": [
					"interpolate",
					["linear"],
					["zoom"],
					7,
					0,
					8,
					1,
				],
			},
		},
		"waterway"
	);
};

//province
const activateProvinceLayer = () => {
	map.addSource("provinces", {
		type: "geojson",
		data: "https://api.maptiler.com/data/0ec90160-ef83-4c01-9a57-e6eb62169af5/features.json?key=aJ1aWBQyxkpsVjqWZPpk",
	});
	map.addLayer({
		id: "provinces",
		type: "line",
		source: "provinces",
		layout: {},
		paint: {
			"line-color": "rgb(68, 138, 255)",
			"line-width": 3,
		},
	});
};
const disableProvinceLayer = () => {
	map.removeLayer("provinces");
	map.removeSource("provinces");
};

//marker
const activateMarker = () => {
	// Marker
	data = jsonData;

	for (let i = 0; i < 1000; i++) {
		if (data[i]["data-geo-point"] != null) {
			let geoPoint = JSON.parse(data[i]["data-geo-point"]);

			// console.log(data[i]);
			var popup = new maplibregl.Popup({ offset: 25 }).setHTML(
				`<a>${data[i]["link"]}</a> <br> Rp.<span>${data[i]["data-price"]}</span>`
			);
			// markers = [];
			var marker = new maplibregl.Marker({ color: "#FF0000" })
				.setLngLat(geoPoint)
				.setPopup(popup)
				.addTo(map);

			markers.value.push(marker);
			console.log(markers);
		}
	}
};
const disableMarker = () => {
	markers.value.forEach((marker) => marker.remove());
	// console.log(markers);
};

//geopoint
const activateGeopoint = () => {
	map.addSource("south-tangerang", {
		type: "geojson",
		data: "https://api.maptiler.com/data/cf974d44-835d-410f-9198-a70a3aa76178/features.json?key=aJ1aWBQyxkpsVjqWZPpk",
	});
	map.loadImage(
		"https://img.icons8.com/plasticine/100/000000/marker.png",
		function (error, image) {
			if (error) throw error;
			map.addImage("custom-marker", image);
		}
	);

	map.addLayer({
		id: "south-tangerang",
		type: "symbol",
		source: "south-tangerang",
		layout: {
			"icon-image": "custom-marker",
			"icon-size": 0.4,
		},
	});
};

const disableGeopoint = () => {
	map.removeLayer("south-tangerang");
	map.removeSource("south-tangerang");
};

//FO layer
const activateFOLayer = () => {
	map.addSource("sp_data_fo_all_190422", {
		type: "vector",
		tiles: [
			`https://pmt.kominfo.go.id/panel/mvt/sp_data_fo_all_190422?z={z}&x={x}&y={y}&access_token=${token.value}`,
		],
	});

	map.addLayer({
		id: "sp_data_fo_all_190422",
		type: "line",
		"source-layer": "sp_data_fo_all_190422",
		source: "sp_data_fo_all_190422",
		paint: {
			"line-color": "#F36A1D",
			"line-width": 2,
			"line-opacity": [
				"case",
				["boolean", ["feature-state", "hover"], false],
				0.8,
				0.4,
			],
		},
	});
};
const disableFOLayer = () => {
	map.removeLayer("sp_data_fo_all_190422");
	map.removeSource("sp_data_fo_all_190422");
};

const activateBTSLayer = () => {
	map.addSource("sp_data_c_bts_3g_tsel_q3_2020", {
		type: "vector",
		tiles: [
			`https://pmt.kominfo.go.id/panel/mvt/sp_data_c_bts_3g_tsel_q3_2020?z={z}&x={x}&y={y}&access_token=${token.value}`,
		],
		cluster: true,
		clusterMaxZoom: 14, // Max zoom to cluster points on
		clusterRadius: 50,
	});

	//image
	map.loadImage(
		"https://img.icons8.com/external-vitaliy-gorbachev-flat-vitaly-gorbachev/58/000000/external-signal-tower-radio-vitaliy-gorbachev-flat-vitaly-gorbachev.png",
		function (error, image) {
			if (error) throw error;
			map.addImage("antena", image);
		}
	);
	map.addLayer({
		id: "sp_data_c_bts_3g_tsel_q3_2020",
		type: "symbol",
		source: "sp_data_c_bts_3g_tsel_q3_2020",
		"source-layer": "sp_data_c_bts_3g_tsel_q3_2020",
		layout: {
			"icon-image": "antena",
			"icon-size": 0.5,
		},
	});
};
</script>

<style>
.map-wrap {
	position: relative;
	width: 100%;
	height: calc(
		100vh - 77px
	); /* calculate height of the screen minus the heading */
}

.map {
	position: absolute;
	width: 100%;
	height: 100%;
}
button {
	margin: 0 0.3rem 0 0;
}
</style>
