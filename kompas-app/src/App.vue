<script setup>
import { ref, onMounted } from 'vue';

const sirka = ref(null)
const delka = ref(null)
const chyba = ref(null)
const vzdalenost = ref(null)
const azimunt = ref(null)

const cil = ref(null)

function spocitejvzdalenost(lat1, lon1, lat2, lon2){
    const R = 6371e3 //polomer zeme
    const f1 = lat1 * Math.PI / 180
    const f2 = lat2 * Math.PI / 180
    const df = (lat2 - lat1) * Math.PI / 180
    const dl = (lon2 - lon1) * Math.PI / 180

    const a = Math.sin(df / 2) * Math.sin(df / 2) + Math.cos(f1) * Math.cos(f2) * Math.sin(dl / 2) * Math.sin(dl / 2)
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))

    return Math.round(R * c)
}

function spocitejAzimut(lat1, lon1, lat2, lon2){
    const f1 = lat1 * Math.PI / 180
    const f2 = lat2 * Math.PI / 180
    const dl = (lon2 - lon1) / Math.PI / 180

    const y = Math.sin(dl) * Math.cos(f2)
    const x = Math.cos(f1) * Math.sin(f2) - Math.sin(f1) * Math.cos(f2) * Math.cos(dl)

    let uhel = Math.atan2(y, x) * 180 / Math.PI
    return Math.round((uhel + 360) % 360)
}

async function nejdiNejblizsiCil(lat, lon) {
    const query = `
    [out:json];
    node["amenity"="pub"](around:2000, ${lat}, ${lon});
    out 1;
    `

    const url = `https://overpass-api.de/api/interpreter?data=${encodeURIComponent(query)}`

    try {
        const odpoved = await fetch(url)
        const data = await odpoved.json()

        if(data.elements.length > 0) {
            cil.value = data.elements[0]

            vzdalenost.value = spocitejvzdalenost(lat, lon, cil.value.lat, cil.value.lon)
            azimunt.value = spocitejAzimut(lat, lon, cil.value.lat, cil.value.lon)
        }else{
            chyba.value = "V okruhu 2 km není žádná hospoda."
        } 
    } catch (err){
        chyba.value = "Chyba sítě při stahování dat."
    }
}

onMounted( () => {
    if(!navigator.geolocation){
        chyba.value = "Geolokace není podporována."
        return
    }

    navigator.geolocation.getCurrentPosition(
        (pozice) => {
            sirka.value = pozice.coords.latitude
            delka.value = pozice.coords.longitude
            
            // TOTO JSI VYNECHAL - Spuštění hledání na mapě:
            nejdiNejblizsiCil(sirka.value, delka.value)
        },
        (err) => {
            chyba.value = err.message
        }
    )
})
</script>

<template>
    <main>
    <h1>Kompas: Hledám cíl</h1>
    
    <p v-if="chyba" style="color: red;">Chyba: {{ chyba }}</p>
    
    <div v-else-if="sirka && delka">
    <p>Moje poloha: {{ sirka }}, {{ delka }}</p>

    <div v-if="cil">
        <h2>Cíl nalezen!</h2>
        <p><strong>Název:</strong> {{ cil.tags?.name || 'Hospoda bez jména' }}</p>
        <p><strong>Souřadnice cíle:</strong> {{ cil.lat }}, {{ cil.lon }}</p>

        <p><strong>Vzdálenost:</strong> {{ vzdalenost }} metrů</p>
        <p><strong>Azimunt:</strong> {{ azimunt }}°</p>
    </div>
    <p v-else>Hledám nejbližší hospodu v OpenStreetMap...</p>

    </div>
    
    <p v-else>Čekám na GPS signál...</p>
    </main>
</template>