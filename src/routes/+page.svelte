<script lang="ts">
    import List from "./List.svelte";
    import { wIsLoading } from '$lib/stores';

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
    let noResult = false;
    const getDongArray = async(dongName:string) => {
        dongArray = [];
        noResult = false;
        let url = `https://apis.zigbang.com/v2/search?leaseYn=N&q=${dongName}&serviceType=원룸`;
        let response = await fetch(url);
        let data = await response.json();
        dongArray = data.items;
        if(dongArray.length === 0) {
            noResult = true;
        }
    }

    let addrDong = '';
</script>
<div class="wrap">
    <small>지자체명을 같이 넣으면 검색이 더 용이합니다.<br>(예: 전주 효자)</small>
    <form>
        <div class="input-group">
            <input type="text" placeholder="동이름 입력" bind:value={addrDong}/>
            <button type="submit" class="btn" onclick={() => getDongArray(addrDong)}>검색</button>
        </div>
    </form>
    <div id="refreshBox">
        <button onclick={
            () => {
                addrDong = '';
                dongArray = [];
            }
        }><i class="xi xi-2x xi-refresh"></i></button>
    </div>
    <div>
        {#if dongArray.length > 0}
            {#each dongArray as dong}
                <List {dong} />
            {/each}
        {/if}
        {#if noResult}
        <div id="msg">검색결과가 없습니다.</div>
        {/if}
    </div>
    {#if $wIsLoading}
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
        text-align: center;
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
    #refreshBox {
        display: flex;
        justify-content: center;
        margin-top:10px;
    }
    #refreshBox button {
        border:solid #ccc 1px;
        border-radius: 15px;
    }
    .input-group {
        display: flex;
        justify-content: center;
        align-items: center;
        gap:5px;
        margin-top:5px;
    }
    input {
        font-size: 1.25rem;
        line-height: 2rem;
        text-align: center;
    }
    .btn {
        font-size: 1.45rem;
    }
    #msg {
        text-align: center;
        font-size: 0.85rem;
        color: #999;
        padding:15px 0;
    }
</style>