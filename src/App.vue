<template>
<div class="app-root" v-cloak>
<header>
  <div class="bar">
    <div class="brand" @click="go('home')">
      <span class="mark"><svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
        <circle cx="14" cy="4.6" r="2.1" fill="#fff"/>
        <g stroke="#fff" stroke-width="1.95" stroke-linecap="round" stroke-linejoin="round">
          <path d="M13.4 7.7 L11.7 12.7"/>
          <path d="M11.7 12.7 L8.7 19.2"/>
          <path d="M11.7 12.7 L15.2 14.4 L14.5 19.6"/>
          <path d="M12.3 9.3 L9.1 11"/>
          <path d="M12.5 9.5 L15.7 11.2"/>
        </g>
      </svg></span>
      <span>마실<br><span class="sub">로그인 없는 지역 나들이</span></span>
    </div>
    <div class="tabs" role="tablist">
      <button class="tab" :class="{on:view==='home'}" @click="go('home')">홈</button>
      <button class="tab" :class="{on:view==='map'}"  @click="go('map')">지도</button>
      <button class="tab" :class="{on:view==='cal'}"  @click="go('cal')">캘린더</button>
      <button class="tab" :class="{on:view==='comm'}" @click="go('comm')">커뮤니티</button>
    </div>
  </div>
</header>

<main>
  <!-- 홈 -->
  <section class="view" v-show="view==='home'">
    <div class="hero" @mouseenter="heroPause()" @mouseleave="heroPlay()">
      <div class="hero-bg" v-for="(s,i) in heroSlides" :key="i" :class="{on:heroIndex===i}"
        :style="s.img?{backgroundImage:'url('+s.img+')'}:{}"></div>
      <div class="hero-ov"></div>
      <button class="hero-arrow left" @click="heroPrev()" aria-label="이전 배너">‹</button>
      <button class="hero-arrow right" @click="heroNext()" aria-label="다음 배너">›</button>
      <transition name="hfade" mode="out-in">
        <div class="hero-inner" :key="heroIndex">
          <template v-if="cur.region===''">
            <div class="kicker">계정 없이 · 알고리즘 없이 · 그냥 지도로</div>
            <h1>동네 이야기는 피드가 아니라 <em>지도</em>에 있어야 합니다.</h1>
            <p>가볼 곳·축제·맛집 정보가 SNS 계정과 알고리즘 안에 갇혀 있어요. 마실은 <b>로그인도, 팔로우도 필요 없이</b> 전국 정보를 지도와 달력으로 펼쳐 봅니다.</p>
            <div class="cta">
              <button class="btn btn-p" @click="heroMap('')">🗺️ 지도로 둘러보기</button>
              <button class="btn btn-g" @click="heroCal('')">📅 축제 일정 보기</button>
            </div>
          </template>
          <template v-else>
            <div class="kicker">권역 둘러보기 · 대표 명소 {{ cur.place }}</div>
            <h1>{{ REG_NAME[cur.region] }}, <em>오늘 어디로</em> 마실 갈까요?</h1>
            <p>가볼 곳 <b>{{ STATS.byreg[cur.region].toLocaleString() }}곳</b> · 축제 <b>{{ festByReg[cur.region] }}건</b>. {{ REG_NAME[cur.region] }}의 명소와 일정을 지도·달력으로 만나보세요.</p>
            <div class="cta">
              <button class="btn btn-p" @click="heroMap(cur.region)">🗺️ {{ REG_NAME[cur.region] }} 지도로 둘러보기</button>
              <button class="btn btn-g" @click="heroCal(cur.region)">📅 {{ REG_NAME[cur.region] }} 축제 일정 보기</button>
            </div>
          </template>
        </div>
      </transition>
      <div class="hero-dots">
        <button v-for="(s,i) in heroSlides" :key="i" :class="{on:heroIndex===i}" @click="heroSet(i)" :aria-label="'슬라이드 '+(i+1)"></button>
      </div>
    </div>
    <div class="why">
      <div class="card"><div class="t">왜 마실을 만들었을까요?</div><h3>지역 정보가 SNS 안에 갇혀 있습니다.</h3>
        <p>동네 정보를 찾거나 경험을 공유하려면 인스타그램이나 블로그 같은 SNS를 이용해야 합니다. 로그인과 알고리즘이 전제되는 구조라, SNS를 사용하지 않으면 정보를 찾기도 나누기도 쉽지 않습니다.</p></div>
      <div class="card"><div class="t">마실이란?</div><h3>로그인 없는 공용 공간</h3>
        <p>계정 생성이나 피드에 지칠 필요 없습니다. 누구나 지역 정보를 확인하고 익명으로 경험을 공유할 수 있는 가장 가벼운 지역 커뮤니티입니다.</p></div>
    </div>
    <div class="stat-row">
      <div class="stat"><div class="n">{{ STATS.total.toLocaleString() }}</div><div class="l">등록된 장소</div></div>
      <div class="stat"><div class="n">{{ REG_NAME.length }}</div><div class="l">개 권역</div></div>
      <div class="stat"><div class="n">{{ CAT_NAME.length }}</div><div class="l">개 카테고리</div></div>
      <div class="stat"><div class="n">0</div><div class="l">필요한 로그인</div></div>
    </div>
    <div class="panels">
      <div class="panel"><h3>카테고리별 장소 <span class="s">{{ donutRegion===''?'전국 합계':REG_NAME[donutRegion]+' 기준' }}</span></h3>
        <div class="dtabs">
          <button :class="{on:donutRegion===''}" @click="donutRegion=''">전국 합계</button>
          <button v-for="(r,i) in REG_NAME" :key="i" :class="{on:donutRegion===i}" @click="donutRegion=i">{{ r }}</button>
        </div>
        <div class="donut">
          <svg class="donut-svg" viewBox="0 0 160 160">
            <circle cx="80" cy="80" r="60" fill="none" stroke="#F1F1F5" stroke-width="26"/>
            <circle class="seg" v-for="(s,i) in donut" :key="i" cx="80" cy="80" r="60" fill="none"
              :stroke="s.color" :stroke-width="hoverCat===i?30:26" :stroke-dasharray="s.dash" :stroke-dashoffset="s.off"
              transform="rotate(-90 80 80)" :style="{cursor:'pointer',opacity:hoverCat===-1||hoverCat===i?1:.4}"
              @click="showCategory(i)" @mousemove="tipMove($event,i)" @mouseleave="tipHide()"/>
            <text x="80" y="75" text-anchor="middle" class="dc-n">{{ donutTotal.toLocaleString() }}</text>
            <text x="80" y="93" text-anchor="middle" class="dc-l">{{ donutRegion===''?'전체 장소':REG_NAME[donutRegion] }}</text>
          </svg>
          <div class="dlegend">
            <div class="dl" v-for="(name,i) in CAT_NAME" :key="i" :class="{hl:hoverCat===i}" style="cursor:pointer"
              @click="showCategory(i)" @mouseenter="hoverCat=i" @mouseleave="hoverCat=-1">
              <span class="dot" :style="{background:CAT_COLOR[i]}"></span>
              <span class="dn">{{ name }}</span>
              <span class="dv">{{ donutCounts[i].toLocaleString() }}<em>{{ pctCat(i) }}%</em></span>
            </div>
          </div>
        </div>
        <div class="dhint">💡 조각이나 범례를 누르면 그 카테고리만 지도에서 볼 수 있어요</div>
        </div>
      <div class="panel"><h3>권역별 장소 <span class="s">막대 길이=장소 수 · 색=카테고리</span></h3>
        <div class="rstack" v-for="(rname,ri) in REG_NAME" :key="ri" :class="{hl:hoverReg===ri}"
          @click="showRegion(ri)" @mouseenter="hoverReg=ri" @mouseleave="hoverReg=-1;tipHide()">
          <div class="rl">{{ rname }}</div>
          <div class="rtrack">
            <div class="rseg" v-for="s in regSegs(ri)" :key="s.ci"
              :style="{width:(animReg?s.count/maxReg*100:0)+'%',background:s.color}"
              @mousemove.stop="tipRegMove($event,ri,s.ci)"></div>
          </div>
          <div class="rn">{{ STATS.byreg[ri].toLocaleString() }}</div>
        </div>
        <div class="rhint">💡 권역을 누르면 지도에서 그 지역 전체를 볼 수 있어요</div>
        </div>
    </div>
    <div class="navcards">
      <button class="navcard" @click="go('map')"><div class="ico" style="background:var(--brand-soft);color:var(--brand-ink)">🗺️</div>
        <h3>지도</h3><p>전국 {{ STATS.total.toLocaleString() }}곳을 카테고리별 색으로 한눈에. 지역·종류로 걸러 찾아보세요.</p></button>
      <button class="navcard" @click="go('cal')"><div class="ico" style="background:#FDECEF;color:#c93a63">📅</div>
        <h3>캘린더</h3><p>날짜가 있는 축제·행사를 달력에 펼쳐서. 이번 주말 뭐가 열리는지 바로.</p></button>
      <button class="navcard" @click="go('comm')"><div class="ico" style="background:#E9F7F0;color:#0c7d55">💬</div>
        <h3>커뮤니티</h3><p>로그인 없이 익명으로. 비밀번호로 내 글을 수정·삭제해요.</p></button>
    </div>
  </section>

  <!-- 지도 -->
  <section class="view" v-show="view==='map'">
    <div class="view-head"><h2>지도에서 찾기</h2><span class="sub">점을 누르면 상세 정보가 열려요</span></div>
    <div class="toolbar">
      <select class="reg" v-model="selRegion">
        <option value="">전체 지역</option>
        <option v-for="(r,i) in REG_NAME" :key="i" :value="i">{{ r }}</option>
      </select>
      <input class="search" v-model="q" placeholder="이름으로 검색…">
      <div class="chips">
        <button class="chip" v-for="(name,i) in CAT_NAME" :key="i" :class="cats.includes(i)?'on':'off'" @click="toggleCat(i)">
          <span class="dot" :style="{background:CAT_COLOR[i]}"></span>{{ name }}
        </button>
      </div>
    </div>
    <div class="map-wrap">
      <div id="map"></div>
      <div class="list">
        <div class="list-head" v-if="filtered.length">총 {{ filtered.length.toLocaleString() }}곳<span v-if="filtered.length>150"> · 150곳 표시</span></div>
        <div class="list-empty" v-if="!filtered.length">조건에 맞는 장소가 없어요</div>
        <button class="fcard" v-for="d in visiblePlaces" :key="d.x" @click="openDetail(d)">
          <img class="thumb" v-if="d.im" :src="d.im" loading="lazy" alt="">
          <span class="thumb" v-else></span>
          <span class="tx">
            <span class="badge" :style="{background:CAT_COLOR[d.c]}">{{ CAT_NAME[d.c] }}</span>
            <h3>{{ d.t }}</h3>
            <div class="fmeta">{{ REG_NAME[d.r] }} <span v-if="d.g">· {{ d.g }}</span></div>
          </span>
        </button>
      </div>
    </div>
  </section>

  <!-- 캘린더 -->
  <section class="view" v-show="view==='cal'">
    <div class="view-head"><h2>일정 한눈에 보기</h2><span class="sub">막대를 누르면 상세 정보가 열려요</span></div>
    <div class="calzones">
      <button :class="{on:calRegion===''}" @click="calRegion=''">전체</button>
      <button v-for="(r,i) in REG_NAME" :key="i" :class="{on:calRegion===i}" @click="calRegion=i">
        <span class="d" :style="{background:REGION_COLOR[i]}"></span>{{ r }}
      </button>
    </div>
    <div class="cal-top">
      <button class="navbtn" @click="calMove(-1)" aria-label="이전 달">‹</button>
      <button class="navbtn" @click="calMove(1)" aria-label="다음 달">›</button>
      <div class="cal-title"><span class="mono">{{ calY }}</span>{{ calM+1 }}월</div>
      <button class="today-btn" @click="calToday">오늘</button>
    </div>
    <div class="cal-scroll"><div class="cal">
      <div class="cal-week"><div class="wd" :class="{sun:i===0}" v-for="(d,i) in WD" :key="i">{{ d }}</div></div>
      <div class="cal-body">
        <div class="cal-grid">
          <div class="cal-cell" v-for="(c,i) in calCells" :key="i" :class="{out:c.out,sun:c.sun,today:c.today}"><span class="daynum">{{ c.day }}</span></div>
        </div>
        <div class="bars">
          <button class="cbar" v-for="(b,i) in calBars.bars" :key="'b'+i" :class="{'cont-l':b.cont,'cont-r':b.contR}" :style="b.style" :title="b.name" @click="openDetail(b.fest)">
            <span class="cbar-dot"></span><span class="cbar-name">{{ b.name }}</span>
          </button>
          <div class="ovf" v-for="(o,i) in calBars.overs" :key="'o'+i" :style="o.style">{{ o.text }}</div>
        </div>
      </div>
    </div></div>
    <div class="cal-note">
      <span v-if="calRegion===''">축제·행사 {{ FESTS.length }}건 (전국, TourAPI 수집). 색은 지역 기준.</span>
      <span v-else>{{ REG_NAME[calRegion] }} {{ calFests.length }}건만 표시 중 · 전체를 보려면 위 「전체」를 누르세요.</span>
    </div>
    <div class="cal-legend"><span class="lg" v-for="(r,i) in REG_NAME" :key="i"><span class="dot" :style="{background:REGION_COLOR[i]}"></span>{{ r }}</span></div>
  </section>

  <!-- 커뮤니티 -->
  <section class="view" v-show="view==='comm'">
    <div class="view-head"><h2>커뮤니티</h2><span class="sub">로그인 없이 · 지역별 익명 게시판</span></div>
    <div class="comm">
      <div class="calzones">
        <button :class="{on:commRegion===''}" @click="commRegion=''">전체</button>
        <button v-for="(r,i) in REG_NAME" :key="i" :class="{on:commRegion===i}" @click="commRegion=i">
          <span class="d" :style="{background:REGION_COLOR[i]}"></span>{{ r }}
        </button>
      </div>
      <div class="composer">
        <div class="av-pick">
          <span class="av-preview" :style="{background:avBg(form.avatar)}"><img :src="avSrc(form.avatar)" alt=""></span>
          <div class="av-grid">
            <button v-for="(src,i) in AVATARS" :key="i" type="button" class="av-opt" :class="{on:form.avatar===i}"
              :style="{background:avBg(i)}" @click="form.avatar=i" :aria-label="'프로필 캐릭터 '+(i+1)" :aria-pressed="form.avatar===i"><img :src="src" alt=""></button>
          </div>
        </div>
        <div class="row">
          <input class="nick" v-model="form.nick" maxlength="16" placeholder="닉네임 (선택)">
          <select class="reg" v-model="form.region" style="max-width:168px">
            <option value="">자유 (전체)</option>
            <option v-for="(r,i) in REG_NAME" :key="i" :value="i">{{ r }} 게시판</option>
          </select>
        </div>
        <textarea v-model="form.body" maxlength="500" placeholder="다녀온 곳, 궁금한 점, 같이 갈 사람… 무엇이든 좋아요"></textarea>
        <div class="row" style="margin-top:10px">
          <input class="pw" v-model="form.pw" type="password" maxlength="20" placeholder="삭제·수정용 비밀번호 (필수)">
        </div>
        <div class="foot">
          <button class="post-btn" :disabled="!form.body.trim()||!form.pw.trim()" @click="createPost">등록</button>
          <span class="hint">비밀번호는 이 글을 수정·삭제할 때 필요해요</span>
        </div>
      </div>
      <div class="store-note">지금은 이 브라우저에만 저장돼요(localStorage). 비밀번호는 기기 밖으로 나가지 않습니다.</div>
      <div class="feed">
        <div class="feed-empty" v-if="!filteredPosts.length"><b>아직 이야기가 없어요</b>이 게시판의 첫 글을 남겨보세요.</div>
        <div class="post" v-for="p in filteredPosts" :key="p.id" @click="openPost(p)">
          <div class="ph">
            <span class="pavatar" :style="{background:avBg(p.avatar)}"><img :src="avSrc(p.avatar)" alt=""></span>
            <span class="rbadge" :style="{background:(p.region===''||p.region==null)?'#8A8A96':REGION_COLOR[p.region]}">{{ boardName(p.region) }}</span>
            <span class="nick" :class="{anon:!p.nick}">{{ p.nick || '익명' }}</span>
            <span class="time">{{ relTime(p.created) }}<span v-if="p.updated"> · 수정됨</span></span>
            <span class="lock">🔒</span>
          </div>
          <div class="body">{{ p.body }}</div>
        </div>
      </div>
    </div>
  </section>
</main>

<!-- 장소 상세 모달 -->
<div class="modal-bg" v-if="detail" @click.self="detail=null">
  <div class="modal modal-split" :class="{'no-media':!detail.im}">
    <button class="m-close" @click="detail=null">×</button>
    <div class="m-media" v-if="detail.im" :style="{background:CAT_COLOR[detail.c]}">
      <img :src="detail.im" alt="">
    </div>
    <div class="m-side">
      <div class="m-side-head">
        <span class="mbadge" :style="{background:CAT_COLOR[detail.c]}">{{ CAT_NAME[detail.c] }}</span>
        <h3>{{ detail.t }}</h3>
      </div>
      <div class="m-body">
        <div class="m-row"><span class="k">지역</span><span class="v">{{ REG_NAME[detail.r] }} {{ detail.g }}</span></div>
        <div class="m-row" v-if="detail.s"><span class="k">기간</span><span class="v mono">{{ detail.s }} ~ {{ detail.e }}</span></div>
        <div class="m-row" v-if="detail.te"><span class="k">문의</span><span class="v">{{ detail.te }}</span></div>
        <div class="m-row"><span class="k">주소</span><button class="v m-addr" @click="openLoc(detail)">{{ detail.a || '-' }} ↗</button></div>
        <button class="m-map" @click="openGmap(detail)">지도앱에서 길찾기</button>
      </div>
    </div>
  </div>
</div>

<!-- 위치 미니지도 -->
<div class="modal-bg" v-if="loc" @click.self="loc=null">
  <div class="locpop">
    <div class="lh"><div><div class="lt">{{ loc.t }}</div><div class="la">{{ REG_NAME[loc.r] }} {{ loc.g }} · {{ loc.a }}</div></div>
      <button class="x" @click="loc=null">×</button></div>
    <div id="miniMap"></div>
  </div>
</div>

<!-- 커뮤니티 상세 모달 -->
<div class="modal-bg" v-if="postView" @click.self="closePost()">
  <div class="modal">
    <button class="m-close" style="background:rgba(20,20,25,.32)" @click="closePost()">×</button>
    <div class="pm-head">
      <span class="rbadge" :style="{background:(postView.region===''||postView.region==null)?'#8A8A96':REGION_COLOR[postView.region],display:'inline-block',marginBottom:'9px'}">{{ boardName(postView.region) }}</span>
      <div class="pm-who">
        <span class="pm-avatar" :style="{background:avBg(postView.avatar)}"><img :src="avSrc(postView.avatar)" alt=""></span>
        <div>
          <div class="nick" :class="{anon:!postView.nick}">{{ postView.nick || '익명' }}</div>
          <div class="time">{{ fullTime(postView.created) }}<span v-if="postView.updated"> · 수정 {{ relTime(postView.updated) }}</span></div>
        </div>
      </div>
    </div>
    <template v-if="!editing">
      <div class="pm-body">{{ postView.body }}</div>
      <div class="pm-actions" v-if="!pwMode">
        <button class="abtn edit" @click="askPw('edit')">수정</button>
        <button class="abtn del" @click="askPw('del')">삭제</button>
      </div>
      <div class="pwbox" v-if="pwMode">
        <div class="lab">{{ pwMode==='edit'?'수정':'삭제' }}하려면 글 작성 때 정한 비밀번호를 입력하세요</div>
        <div class="prow">
          <input type="password" v-model="pwInput" placeholder="비밀번호" @keyup.enter="confirmPw()">
          <button class="go" @click="confirmPw()">확인</button>
        </div>
        <div class="err" v-if="pwErr">비밀번호가 일치하지 않아요</div>
      </div>
    </template>
    <div class="edit-area" v-else>
      <div class="av-pick" style="margin-bottom:9px">
        <span class="av-preview" :style="{background:avBg(edit.avatar)}"><img :src="avSrc(edit.avatar)" alt=""></span>
        <div class="av-grid">
          <button v-for="(src,i) in AVATARS" :key="i" type="button" class="av-opt" :class="{on:edit.avatar===i}"
            :style="{background:avBg(i)}" @click="edit.avatar=i" :aria-label="'프로필 캐릭터 '+(i+1)" :aria-pressed="edit.avatar===i"><img :src="src" alt=""></button>
        </div>
      </div>
      <input v-model="edit.nick" maxlength="16" placeholder="닉네임 (선택)">
      <textarea v-model="edit.body" maxlength="500"></textarea>
      <button class="save" @click="saveEdit()">저장</button>
    </div>
  </div>
</div>

<!-- 도넛 툴팁 -->
<div class="dtip" v-if="tip.show" :style="{left:tip.x+'px',top:tip.y+'px'}">
  <b>{{ tip.name }}</b> {{ tip.val }}곳 · {{ tip.pct }}%
</div>

<!-- 챗봇 -->
<chat-widget></chat-widget>
</div>
</template>

<script>
import L from 'leaflet'
import ChatWidget from './components/ChatWidget.vue'
import {
  DATA, STATS, CAT_COLOR, REGION_COLOR, AVATARS, AV_BG,
  avIdx, avSrc, avBg, ZONES, GU2ZONE, FESTS, REGCAT, REGBOUNDS,
  hashPw, SAMPLE_POSTS, FEST_BY_REG, HERO,
} from './lib.js'

export default {
  name: 'App',
  components: { ChatWidget, 'chat-widget': ChatWidget },
  data(){return{
    view:'home',
    CAT_NAME:STATS.cats, REG_NAME:STATS.regions, CAT_COLOR, REGION_COLOR, AVATARS, STATS, ZONES, FESTS, WD:['일','월','화','수','목','금','토'],
    heroSlides:HERO, heroIndex:0, heroTimer:null, festByReg:FEST_BY_REG,
    // 지도
    mapReady:false, selRegion:'', cats:[0,1,2,3,4,6,7], q:'',
    detail:null, loc:null,
    tip:{show:false,x:0,y:0,name:'',val:'',pct:''}, hoverCat:-1, donutRegion:'', hoverReg:-1, animReg:false,
    // 캘린더
    calY:2026, calM:6, calRegion:'',
    // 커뮤니티
    posts:JSON.parse(localStorage.getItem('masil_posts')||'[]'),
    commRegion:'', form:{nick:'',body:'',pw:'',region:'',avatar:0},
    postView:null, editing:false, pwMode:'', pwInput:'', pwErr:false, edit:{nick:'',body:'',avatar:0},
    _t:0,
  }},
  computed:{
    cur(){ return this.heroSlides[this.heroIndex]; },
    maxCat(){ return Math.max(...STATS.bycat); },
    maxReg(){ return Math.max(...STATS.byreg); },
    donutCounts(){ return this.donutRegion===''?STATS.bycat:REGCAT[this.donutRegion]; },
    donutTotal(){ return this.donutCounts.reduce((a,b)=>a+b,0); },
    donut(){
      const C=2*Math.PI*60, total=this.donutTotal||1, GAP=1.3; let cum=0; const out=[];
      this.donutCounts.forEach((v,i)=>{
        const len=v/total*C, draw=v>0?Math.max(len-GAP,0.6):0;
        out.push({color:CAT_COLOR[i], dash:draw+' '+(C-draw), off:-cum});
        cum+=len;
      });
      return out;
    },
    filtered(){
      const q=this.q.trim().toLowerCase();
      return DATA.filter(d=> this.cats.includes(d.c)
        && (this.selRegion===''||d.r==this.selRegion)
        && (!q||d.t.toLowerCase().includes(q)) );
    },
    visiblePlaces(){ return this.filtered.slice(0,150); },
    filteredPosts(){
      const list=this.commRegion===''?this.posts:this.posts.filter(p=>p.region===this.commRegion);
      return list.slice().sort((a,b)=>b.created-a.created);
    },
    calFests(){ return this.calRegion===''?FESTS:FESTS.filter(f=>f.r===this.calRegion); },
    calCells(){
      const y=this.calY,mo=this.calM, startDay=new Date(y,mo,1).getDay();
      const today=new Date(); today.setHours(0,0,0,0);
      const out=[];
      for(let i=0;i<42;i++){ const d=new Date(y,mo,1+(i-startDay));
        out.push({day:d.getDate(),out:d.getMonth()!==mo,sun:d.getDay()===0,today:d.getTime()===today.getTime()}); }
      return out;
    },
    calBars(){
      const y=this.calY,mo=this.calM, startDay=new Date(y,mo,1).getDay();
      const cells=[]; for(let i=0;i<42;i++) cells.push(new Date(y,mo,1+(i-startDay)));
      const CELL_H=132,TOP=32,BH=27; const bars=[],overs=[];
      for(let w=0;w<6;w++){
        const wS=cells[w*7].getTime(), wE=cells[w*7+6].getTime()+86400000-1;
        const segs=[];
        this.calFests.forEach(f=>{
          const fs=new Date(f.s).getTime(), fe=new Date(f.e).getTime()+86400000-1;
          if(fe<wS||fs>wE) return;
          const ss=Math.max(fs,wS), se=Math.min(fe,wE);
          segs.push({f,sCol:new Date(ss).getDay(),eCol:new Date(se).getDay(),cont:fs<wS,contR:fe>wE});
        });
        segs.sort((a,b)=>a.sCol-b.sCol||a.eCol-b.eCol);
        const lanes=[]; segs.forEach(s=>{let l=0;while(l<lanes.length&&lanes[l]>=s.sCol)l++;lanes[l]=s.eCol;s.lane=l;});
        segs.forEach(s=>{ if(s.lane>2) return;
          bars.push({fest:s.f,name:s.f.t,cont:s.cont,contR:s.contR,style:{
            left:'calc('+(s.sCol/7*100)+'% + 4px)', width:'calc('+((s.eCol-s.sCol+1)/7*100)+'% - 8px)',
            top:(w*CELL_H+TOP+s.lane*BH)+'px', background:REGION_COLOR[s.f.r] }});
        });
        const of=segs.filter(s=>s.lane>2), cols={};
        of.forEach(s=>{for(let c=s.sCol;c<=s.eCol;c++)cols[c]=(cols[c]||0)+1;});
        Object.entries(cols).forEach(([c,n])=>overs.push({text:'+'+n,style:{left:'calc('+(c/7*100)+'% + 6px)',top:(w*CELL_H+TOP+3*BH)+'px'}}));
      }
      return {bars,overs};
    },
  },
  watch:{
    view(v){ if(v==='map'){ this.$nextTick(()=>{ this.ensureMap(); this.map.invalidateSize(); this.renderMap(); this.zoomToRegion(); }); } },
    filtered(){ if(this.mapReady) this.renderMap(); },
    selRegion(){ this.zoomToRegion(); },
    commRegion(v){ this.form.region=v; },
    loc(v){ if(v) this.$nextTick(()=>this.buildMini(v)); else if(this.miniMap){ this.miniMap.remove(); this.miniMap=null; } },
  },
  methods:{
    go(v){ this.view=v; window.scrollTo({top:0,behavior:'smooth'}); },
    heroPlay(){ this.heroPause(); this.heroTimer=setInterval(()=>{ this.heroIndex=(this.heroIndex+1)%this.heroSlides.length; },5000); },
    heroPause(){ if(this.heroTimer){ clearInterval(this.heroTimer); this.heroTimer=null; } },
    heroSet(i){ this.heroIndex=i; this.heroPlay(); },
    heroPrev(){ this.heroSet((this.heroIndex-1+this.heroSlides.length)%this.heroSlides.length); },
    heroNext(){ this.heroSet((this.heroIndex+1)%this.heroSlides.length); },
    heroMap(region){ this.selRegion=region; this.cats=[0,1,2,3,4,5,6,7]; this.q=''; this.go('map'); },
    heroCal(region){ this.calRegion=region; this.go('cal'); },
    pctCat(i){ return (this.donutCounts[i]/(this.donutTotal||1)*100).toFixed(1); },
    showCategory(i){ this.cats=[i]; this.q=''; this.selRegion=this.donutRegion; this.tipHide(); this.go('map'); },
    tipMove(e,i){ this.hoverCat=i; this.tip={show:true,x:e.clientX+14,y:e.clientY+16,name:STATS.cats[i],val:this.donutCounts[i].toLocaleString(),pct:this.pctCat(i)}; },
    tipHide(){ this.tip.show=false; this.hoverCat=-1; },
    regSegs(ri){ return CAT_COLOR.map((color,ci)=>({ci,color,count:REGCAT[ri][ci]})).filter(s=>s.count>0); },
    showRegion(ri){ this.selRegion=ri; this.cats=[0,1,2,3,4,5,6,7]; this.q=''; this.tipHide(); this.go('map'); },
    tipRegMove(e,ri,ci){ this.tip={show:true,x:e.clientX+14,y:e.clientY+16,name:STATS.regions[ri]+' · '+STATS.cats[ci],val:REGCAT[ri][ci].toLocaleString(),pct:(REGCAT[ri][ci]/STATS.byreg[ri]*100).toFixed(1)}; },
    toggleCat(i){ const k=this.cats.indexOf(i); if(k>=0) this.cats.splice(k,1); else this.cats.push(i); },
    // 지도
    ensureMap(){
      if(this.mapReady) return; this.mapReady=true;
      this.map=L.map('map',{preferCanvas:true,scrollWheelZoom:true}).setView([36.5,127.8],7);
      L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
        {attribution:'© OpenStreetMap © CARTO',maxZoom:19,subdomains:'abcd'}).addTo(this.map);
      this.markerLayer=L.layerGroup().addTo(this.map); this.renderMap();
    },
    renderMap(){
      if(!this.markerLayer) return;
      this.markerLayer.clearLayers();
      for(const d of this.filtered){
        const m=L.circleMarker([d.la,d.ln],{radius:5,weight:1,color:'#fff',fillColor:CAT_COLOR[d.c],fillOpacity:.92});
        m.on('click',()=>this.openDetail(d)); this.markerLayer.addLayer(m);
      }
    },
    zoomToRegion(){
      if(!this.mapReady) return;
      if(this.selRegion===''||this.selRegion===null){ this.map.setView([36.5,127.8],7,{animate:true}); return; }
      const b=REGBOUNDS[this.selRegion];
      if(b) this.map.fitBounds(b,{padding:[36,36],maxZoom:13,animate:true});
    },
    openDetail(d){ this.detail=d; },
    openGmap(d){ window.open('https://www.google.com/maps/search/?api=1&query='+d.la+','+d.ln,'_blank'); },
    openLoc(d){ this.loc=d; },
    buildMini(d){
      if(this.miniMap){ this.miniMap.remove(); this.miniMap=null; }
      this.miniMap=L.map('miniMap',{zoomControl:true,attributionControl:false}).setView([d.la,d.ln],15);
      L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',{subdomains:'abcd',maxZoom:19}).addTo(this.miniMap);
      L.circleMarker([d.la,d.ln],{radius:8,weight:2,color:'#fff',fillColor:CAT_COLOR[d.c],fillOpacity:1}).addTo(this.miniMap);
      this.miniMap.invalidateSize();
    },
    // 캘린더
    calMove(n){ let m=this.calM+n,y=this.calY; if(m<0){m=11;y--;} if(m>11){m=0;y++;} this.calM=m; this.calY=y; },
    calToday(){ const t=new Date(); this.calY=t.getFullYear(); this.calM=t.getMonth(); },
    // 커뮤니티 CRUD
    savePosts(){ localStorage.setItem('masil_posts',JSON.stringify(this.posts)); },
    createPost(){
      const body=this.form.body.trim(), pw=this.form.pw.trim(); if(!body||!pw) return;
      this.posts.push({id:(this._t=Date.now()+Math.floor(Math.random()*999)),nick:this.form.nick.trim(),avatar:avIdx(this.form.avatar),body,region:this.form.region,pw:hashPw(pw),created:Date.now(),updated:0});
      this.savePosts(); this.form={nick:'',body:'',pw:'',region:this.commRegion,avatar:0};
    },
    avBg(a){ return avBg(a); },
    avSrc(a){ return avSrc(a); },
    boardName(region){ return (region===''||region==null)?'자유':STATS.regions[region]; },
    openPost(p){ this.postView=p; this.editing=false; this.pwMode=''; this.pwInput=''; this.pwErr=false; },
    closePost(){ this.postView=null; this.editing=false; this.pwMode=''; },
    askPw(mode){ this.pwMode=mode; this.pwInput=''; this.pwErr=false; },
    confirmPw(){
      if(hashPw(this.pwInput.trim())!==this.postView.pw){ this.pwErr=true; return; }
      if(this.pwMode==='del'){ this.deletePost(this.postView); }
      else { this.edit={nick:this.postView.nick,body:this.postView.body,avatar:avIdx(this.postView.avatar)}; this.editing=true; this.pwMode=''; }
    },
    saveEdit(){
      const b=this.edit.body.trim(); if(!b) return;
      this.postView.nick=this.edit.nick.trim(); this.postView.avatar=avIdx(this.edit.avatar); this.postView.body=b; this.postView.updated=Date.now();
      this.savePosts(); this.editing=false;
    },
    deletePost(p){ this.posts=this.posts.filter(x=>x.id!==p.id); this.savePosts(); this.closePost(); },
    // 시간
    relTime(ms){ const s=(Date.now()-ms)/1000;
      if(s<60)return'방금'; if(s<3600)return Math.floor(s/60)+'분 전'; if(s<86400)return Math.floor(s/3600)+'시간 전';
      if(s<604800)return Math.floor(s/86400)+'일 전'; const d=new Date(ms); return (d.getMonth()+1)+'.'+d.getDate(); },
    fullTime(ms){ const d=new Date(ms); return d.getFullYear()+'.'+(d.getMonth()+1)+'.'+d.getDate()+' '+String(d.getHours()).padStart(2,'0')+':'+String(d.getMinutes()).padStart(2,'0'); },
  },
  mounted(){ const t=new Date(); this.calY=t.getFullYear(); this.calM=t.getMonth();
    setTimeout(()=>{ this.animReg=true; },200);
    this.heroPlay();
    if(!localStorage.getItem('masil_seeded_v3')){
      const now=Date.now();
      const seed=SAMPLE_POSTS.map((s,k)=>({id:now-k,nick:s.nick,avatar:avIdx(s.avatar),body:s.body,region:s.region,pw:hashPw('1234'),created:now-s.ago*3600000,updated:0}));
      this.posts=seed.concat(this.posts); this.savePosts(); localStorage.setItem('masil_seeded_v3','1');
    }
    document.addEventListener('keydown',e=>{ if(e.key==='Escape'){ this.detail=null; this.loc=null; this.closePost(); } }); },
}
</script>
