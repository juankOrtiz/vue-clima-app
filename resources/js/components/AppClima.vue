<template>
    <div class="text-white mb-8">
        <div class="input-lugares text-gray-800">
            <input type="search" id="direccion" class="w-full" placeholder="Buscar una ciudad">
            <p>Seleccionado: <strong id="valor-direccion">ninguno</strong></p>
        </div>
        <!-- Contenedor general del clima -->
        <div class="contenedor-clima font-sans w-128 max-w-lg overflow-hidden bg-gray-900 shadow-lg mt-4 rounded-lg">
            <!-- Primera parte de la tarjeta con el clima actual -->
            <div class="clima-actual flex items-center justify-between px-6 py-6 ">
                <div class="flex items-center">
                    <!-- Temperatura grande -->
                    <div>
                        <div class="text-6xl font-semibold">{{ temperaturaActual.actual }}°C</div>
                        <div class="">Sensacion termica {{ temperaturaActual.sensacion }}°C</div>
                    </div>
                    <!-- Descripcion clima -->
                    <div class="mx-5">
                        <div class="font-semibold">{{ temperaturaActual.resumen }}</div>
                        <div>{{ localizacion.nombre }}</div>
                    </div>
                </div>
                <!-- Icono -->
                <div>
                    <canvas ref="iconoActual" id="iconoActual" width="96" height="96"></canvas>
                </div>
            </div>
            <!-- Segunda parte de la tarjeta con el pronostico -->
            <div class="pronostico-clima text-sm bg-gray-800 px-6 py-8 overflow-hidden">
                <div v-for="(dia, indice) in diario" 
                    :key="dia.time" 
                    class="flex items-center"
                    :class="{ 'mt-8' : indice > 0 }"
                    v-if="indice < 5"
                >
                    <!-- Dia de la semana -->
                    <div class="w-1/6 text-lg text-gray-200">{{ aDiaDeLaSemana(dia.time) }}</div>
                    <!-- Descripcion del clima -->
                    <div class="w-4/6 px-4 flex items-center">
                        <div>
                            <canvas :id="`icono${indice+1}`" :data-icono="toKebabCase(dia.icon)" width="24" height="24"></canvas>
                        </div>
                        <div class="ml-3">{{ dia.summary }}</div>
                    </div>
                    <!-- Temperaturas -->
                    <div class="w-1/6 text-right">
                        <div>{{ Math.round(dia.temperatureHigh) }}°C</div>
                        <div>{{ Math.round(dia.temperatureLow) }}°C</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        mounted() {
            this.leerDatos()

            // Inicializar Algolia
            const placesAutocomplete = places({
                appId: 'plROI2E2CPC7',
                apiKey: 'bddbb2d5587be9ba4faaff8ac41f0288',
                container: document.querySelector("#direccion"),
            }).configure({
                type: 'city',
                aroundLatLngViaIP: false,
            });

            const $direccion = document.querySelector("#valor-direccion");
            placesAutocomplete.on('change', (e) => {
                $direccion.textContent = e.suggestion.value;

                this.localizacion.nombre = `${e.suggestion.name}, ${e.suggestion.country}`;
                this.localizacion.lat = e.suggestion.latlng.lat;
                this.localizacion.long = e.suggestion.latlng.long;
            });
            placesAutocomplete.on('clear', function() {
                $$direccion.textContent = 'ninguno';
            });
        },
        watch: {
            location: {
                handler(newValue, oldValue) {
                    this.leerDatos()
                },
                deep: true
            }
        },
        data() {
            return {
                temperaturaActual: {
                    actual: '',
                    sensacion: '',
                    resumen: '',
                    icono: '',
                },
                localizacion: {
                    nombre: 'Toronto, Canada',
                    lat: 43.6532,
                    long: -79.38323,
                },
                diario: [],
            }
        },
        methods: {
            leerDatos() {
                const skycons = new Skycons({'color': 'white'});

                fetch(`/api/clima?lat=${this.localizacion.lat}&long=${this.localizacion.long}`)
                .then(response => response.json())
                .then(data => {
                    console.log(data);
                    this.temperaturaActual.actual = Math.round(data.currently.temperature);
                    this.temperaturaActual.sensacion = Math.round(data.currently.apparentTemperature);
                    this.temperaturaActual.resumen = data.currently.summary;
                    this.temperaturaActual.icono = this.toKebabCase(data.currently.icon);

                    this.diario = data.daily.data;

                    skycons.add('iconoActual', this.temperaturaActual.icono);
                    skycons.play();

                    this.$nextTick(() => {
                        skycons.add('icono1', document.getElementById('icono1').getAttribute('data-icono'));
                        skycons.add('icono2', document.getElementById('icono2').getAttribute('data-icono'));
                        skycons.add('icono3', document.getElementById('icono3').getAttribute('data-icono'));
                        skycons.add('icono4', document.getElementById('icono4').getAttribute('data-icono'));
                        skycons.add('icono5', document.getElementById('icono5').getAttribute('data-icono'));
                        skycons.play();
                    });
                })
            },
            toKebabCase(cadenaAConvertir) {
                return cadenaAConvertir.split(' ').join('-');
            },
            aDiaDeLaSemana(timestamp) {
                const nuevaFecha = new Date(timestamp*1000);
                const dias = ['DOM', 'LUN', 'MAR', 'MIE', 'JUE', 'VIE', 'SAB'];
                return dias[nuevaFecha.getDay()];
            }
        }
    }
</script>
