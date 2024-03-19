<script lang="ts">
    import { wIsLoading } from '$lib/stores';
    import * as geohash from 'ngeohash';
    const sleep = (ms: number) => {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    export let dong:any = {};
    let isOpen = false;
    const getOneroom = async(lat: number, lng: number) => {
        isOpen = !isOpen;
        if(!isOpen) {
            infoBox.innerHTML = '';
            return;
        } else {
            wIsLoading.set(true);
            let wrap = document.createElement('div');
            wrap.style.marginBottom = '10px';
            let geo = geohash.encode(lat, lng, 5);
            await sleep(2000);
            let url = `https://apis.zigbang.com/v2/items?deposit_gteq=0&domain=zigbang&geohash=${geo}&needHasNoFiltered=true&rent_gteq=0&sales_type_in=전세|월세&service_type_eq=원룸`;
            let response2 = await fetch(url);
            let data = await response2.json();
            let sections = data.sections;
            for(let section of sections) {
                let strong = document.createElement('strong');
                strong.innerHTML = section.title;
                let span = document.createElement('span');
                span.style.color = 'teal';
                span.innerHTML = ' ('+section.item_ids.length + '개)';
                let li = document.createElement('li');
                li.style.listStyle = 'none';
                li.style.backgroundColor = '#fff';
                li.style.marginBottom = '1px';
                li.style.padding = '10px';
                li.appendChild(strong);
                li.appendChild(span);
                wrap.appendChild(li);
                infoBox.appendChild(wrap);
            }
            wIsLoading.set(false);
        }
    }
    let infoBox: HTMLDivElement;
</script>


<div class="list">
    <a href="https://www.zigbang.com/home/oneroom/map?lat={dong.lat}&lng={dong.lng}&zoom=5" target="_blank">지도보기</a>
    <div class="address">
        {dong.description}
    </div>
    <button onclick={()=>{
        getOneroom(dong.lat, dong.lng);
    }}>
        {dong.name}
        <i class="xi {isOpen?'xi-caret-up-min':'xi-caret-down-min'}"></i>
    </button>
</div>
<div bind:this={infoBox}></div>

<style>
    .list {
        display: flex;
        gap:10px;
        align-items: center;
        border-bottom:solid #fff 1px;
        padding:10px 5px;
    }
    .address {
        flex:1;
    }
    .num {
        color:blue;
    }
    button {
        font-size:1rem;
    }
</style>