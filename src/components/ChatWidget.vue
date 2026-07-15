<template>
    <button class="chat-fab" v-if="!open" @click="toggle" aria-label="챗봇 열기"><span class="fab-bubble">💬</span><span class="fab-bot">🤖</span></button>
    <div class="chat-panel" v-if="open">
      <div class="chat-head">
        <div><div class="t">마실 도우미</div><div class="d">전국 장소 데이터로 답해요 · OpenAI</div></div>
        <button class="ic" @click="showKey=!showKey" title="API 키 설정">🔑</button>
        <button class="ic" @click="clearChat" title="대화 비우기">🗑️</button>
        <button class="ic" @click="toggle" title="닫기">✕</button>
      </div>
      <div class="chat-msgs" ref="msgs">
        <div class="msg sys" v-if="!messages.length">안녕하세요! 가볼 곳·축제·맛집을 물어보세요.<br>예: "성수동 카페 근처 가볼 곳"</div>
        <div v-for="(m,i) in messages" :key="i" class="msg" :class="m.role==='user'?'user':'bot'">{{ m.content }}</div>
        <div class="msg bot" v-if="loading"><span class="dots"><i></i><i></i><i></i></span></div>
      </div>
      <div class="chat-sugg" v-if="!messages.length">
        <button v-for="s in suggestions" :key="s" @click="pick(s)">{{ s }}</button>
      </div>
      <div class="chat-key" v-if="showKey">
        <div class="lab">🔑 OpenAI API 키 (sk-…) · 브라우저에만 저장돼요</div>
        <div class="krow"><input v-model="keyInput" type="password" placeholder="sk-..." @keyup.enter="saveKey"><button class="ks" @click="saveKey">저장</button></div>
      </div>
      <div class="chat-in">
        <input v-model="input" :placeholder="apiKey?'무엇이든 물어보세요…':'먼저 🔑 에서 API 키를 넣어주세요'" @keyup.enter="send">
        <button class="snd" :disabled="loading||!input.trim()" @click="send">전송</button>
      </div>
    </div>
</template>

<script>
import { DATA, STATS } from '../data/masil-data.js'

export default {
  name: 'ChatWidget',
  data(){return{
    open:false, input:'', loading:false,
    apiKey:localStorage.getItem('masil_openai_key')||'',
    keyInput:'', showKey:false,
    messages:JSON.parse(localStorage.getItem('masil_chat')||'[]'),
    suggestions:['종로구 가볼 만한 곳','이번 가을 서울 축제','부산 관광지 추천','아이랑 갈 문화시설'],
  }},
  methods:{
    toggle(){ this.open=!this.open; if(this.open&&!this.apiKey) this.showKey=true; this.$nextTick(this.scrollDown); },
    saveKey(){ this.apiKey=this.keyInput.trim(); localStorage.setItem('masil_openai_key',this.apiKey); this.showKey=false; },
    persist(){ localStorage.setItem('masil_chat',JSON.stringify(this.messages.slice(-40))); },
    scrollDown(){ const el=this.$refs.msgs; if(el) el.scrollTop=el.scrollHeight; },
    clearChat(){ this.messages=[]; this.persist(); },
    pick(s){ this.input=s; this.send(); },
    buildContext(q){
      const toks=q.toLowerCase().split(/\s+/).filter(Boolean);
      let hits=DATA.filter(d=>{
        const hay=(d.t+' '+STATS.cats[d.c]+' '+STATS.regions[d.r]+' '+(d.g||'')).toLowerCase();
        return toks.some(t=>hay.includes(t));
      });
      if(!hits.length) hits=DATA.slice(0,15);
      if(hits.length>28) hits=hits.slice(0,28);
      return JSON.stringify(hits.map(d=>({이름:d.t,종류:STATS.cats[d.c],지역:STATS.regions[d.r]+' '+(d.g||''),주소:d.a,기간:d.s?(d.s+'~'+d.e):undefined})));
    },
    async send(){
      const q=this.input.trim(); if(!q||this.loading) return;
      if(!this.apiKey){ this.showKey=true; return; }
      this.messages.push({role:'user',content:q}); this.input=''; this.persist();
      this.loading=true; this.$nextTick(this.scrollDown);
      try{
        const sys='너는 "마실"의 지역정보 안내 도우미야. 아래 JSON 장소 데이터에 근거해서만 한국어로 친절하고 간결하게 답해. '+
                  '추천할 땐 이름과 지역을 함께 알려줘. 데이터에 없는 건 모른다고 솔직히 말해.\n\n[장소데이터]\n'+this.buildContext(q);
        const hist=this.messages.slice(-6).map(m=>({role:m.role,content:m.content}));
        const res=await fetch('https://api.openai.com/v1/chat/completions',{
          method:'POST',
          headers:{'Content-Type':'application/json','Authorization':'Bearer '+this.apiKey},
          body:JSON.stringify({model:'gpt-4o-mini',temperature:0.4,messages:[{role:'system',content:sys},...hist]})
        });
        const j=await res.json();
        if(!res.ok) throw new Error((j.error&&j.error.message)||('오류 '+res.status));
        this.messages.push({role:'assistant',content:j.choices[0].message.content.trim()});
      }catch(e){
        this.messages.push({role:'assistant',content:'⚠️ '+e.message+'\n(API 키가 올바른지, 사용량이 남아있는지 확인해 주세요. 설정 🔑 에서 키를 다시 넣을 수 있어요.)'});
      }
      this.loading=false; this.persist(); this.$nextTick(this.scrollDown);
    },
  },
}
</script>
