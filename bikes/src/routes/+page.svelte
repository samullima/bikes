<h1>Bem vindo ao meu Projeto</h1>
<p>Bikes</p>

<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
    import { onMount } from "svelte";
    import { readable } from "svelte/store";
    import * as d3 from "d3";

    let mapViewChanged = 0;
    let stations = [];
    let map;
    

    mapboxgl.accessToken = "pk.eyJ1Ijoic2FtdWxsaW1hIiwiYSI6ImNtYXBoYjd6ZzBnaGEybXEydTR6d2E2bGcifQ.FikvxyGGbH9qN2y7spfWgQ";

    onMount(() => {
        let map = new mapboxgl.Map({
            container: 'map',
            style: 'mapbox://styles/mapbox/streets-v12',
            zoom: 12,
            center: [-71.0788727679991, 42.36199480654666]
        });
    })

    async function initMap() {
        map = new mapboxgl.Map({
            container: 'map',
            center: [-71.09415, 42.36027],
            zoom: 12,
            style: "mapbox://styles/mapbox/streets-v12",
        });
        await new Promise(resolve => map.on("load", resolve));
        map.addSource("boston_route", {
            type: "geojson",
            data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
        });
        map.addSource("cambridge_route", {
            type: "geojson",
            data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson"
        });


        map.addLayer({
            id: "SOME_ID", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "boston_route", // The id we specified in `addSource()`
            paint: {
                "line-color": "#FF0000",
            },
        });

        map.addLayer({
            id: "cambridge_route",
            type: "line",
            source: "cambridge_route",
            paint: {
                "line-color": "#4f4f00",
                "line-width": 3,
                "line-opacity": 0.5,
            },
        });
    }

    async function loadStationData() {
        try {
            const csvUrl = 'https://vis-society.github.io/labs/8/data/bluebikes-stations.csv';
            const data = await d3.csv(csvUrl);
            
            stations = data.map(station => ({
                id: station.Number,
                name: station.NAME,
                Lat: +station.Lat,
                Long: +station.Long,
            }));
            // console.log('Stations loaded:', stations.length);
        } catch (error) {
            console.error('Error loading station data:', error);
        }
    }

    function getCoords (station) {
        let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    }

    onMount(() => {
        initMap();
        loadStationData();
    });

    $: map?.on("move", evt => mapViewChanged++);

</script>

<style>
    @import url("../../global.css");
    #map {
        flex: 1;
    }

    #map svg {
        position: absolute;
        z-index: 1;
        width: 100%;
        height: 100%;
        pointer-events: none;
    }
</style>

<div id="map">
    <svg>
        {#key mapViewChanged}
        <!-- render stations here -->
         {#each stations as station}
            <circle { ...getCoords(station) } r="5" fill="steelblue" />
        {/each}

        {/key}

    </svg>
</div>


