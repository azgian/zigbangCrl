<script lang="ts">
    import * as geohash from 'ngeohash';

    function sleep(ms: number) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
function formatPhoneNumber(phoneNumber: string) {
    // 먼저 숫자만 추출합니다.
    let digits = phoneNumber.replace(/\D/g, '');

    // 휴대폰 번호 형식(010, 011 등으로 시작하는 경우)
    if (digits.startsWith('010') || digits.startsWith('011')) {
        return digits.replace(/(\d{3})(\d{3,4})(\d{4})/, '$1-$2-$3');
    }
    // 일반 전화번호 형식
    else {
        return digits.replace(/(\d{2,3})(\d{3,4})(\d{4})/, '$1-$2-$3');
    }
}
let dongArray: any[] = [];
const getDongArray = async(dongName:string) => {
    let infoBox = document.querySelectorAll('.infoBox');
    for(let box of infoBox) {
        box.innerHTML = '';
    }
    let url = `https://apis.zigbang.com/v2/search?leaseYn=N&q=${dongName}&serviceType=원룸`;
    let response = await fetch(url);
    let data = await response.json();
    dongArray = data.items;
}

// 동이름을 넣으면 정보를 반환하는 함수입니다.
let isOpen = false;
let isLoading = false;
async function getOneroom(lat: number, lng: number, infoBox: HTMLDivElement) {
    isOpen = !isOpen;
    if(!isOpen) {
        infoBox.innerHTML = '';
        return;
    }
    isLoading = true;
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
    isLoading = false;
}
let addrDong = '';
</script>
<div class="wrap">
    <form>
    <div class="input-group">
            <input type="text" placeholder="동이름을 입력하세요" bind:value={addrDong}/>
            <button type="submit" onclick={() => getDongArray(addrDong)}>동검색</button>
        </div>
    </form>
    <div>
        {#if dongArray.length > 0}
            {#each dongArray as dong}
                <div class="list">
                    <a href="https://www.zigbang.com/home/oneroom/map?lat={dong.lat}&lng={dong.lng}&zoom=5" target="_blank">지도보기</a>
                    <div class="address">
                        {dong.description}
                    </div>
                    <button onclick={()=>{getOneroom(dong.lat, dong.lng, dong.id)}}>
                        {dong.name}
                    </button>
                </div>
                <div class="infoBox" bind:this={dong.id}></div>
            {/each}
        {/if}
    </div>
    {#if isLoading}
        <div class="cover"><i class="xi xi-3x xi-spin xi-spinner-3"></i></div>
    {/if}
</div>

<style>
    .wrap {
        max-width:600px;
        width:90%;
        margin: 0 auto;
        padding: 20px 10px;
        position:relative;
    }
    .wrap .cover {
        position: absolute;
        top:0;
        left:0;
        width:100%;
        height:100%;
        background:rgba(0,0,0,0.5);
        z-index:100;
        display:flex;
        justify-content:center;
        align-items:center;
        color:rgba(200,200,200,0.5);
    }
    .input-group {
        display: flex;
        justify-content: center;
        align-items: center;
        gap:5px;
        margin-bottom:20px;
    }
    .list {
        display: flex;
        gap:10px;
        align-items: center;
        border-bottom:solid #fff 1px;
        padding:5px;
    }
    .address {
        flex:1;
    }
    .num {
        color:blue;
    }
    input {
        font-size: 1.25rem;
        text-align: center;
    }
    button {
        font-size: 1.15rem;
    }
</style>