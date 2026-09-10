<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'
import {
  ArrowDownRight, ArrowUpRight, BarChart3, Bell, Blocks, Check, Plus,
  ChevronDown, CircleHelp, ExternalLink, Globe2, Menu, MoveRight,
  Radio, ShieldCheck, Sparkles, Wallet, X
} from 'lucide-vue-next'

const activeTab = ref('All')
const connected = ref(false)
const walletAddress = ref('')
const walletError = ref('')
const menuOpen = ref(false)
const toast = ref('')
const pulse = ref(0)
let timer

const markets = ref([
  { symbol: 'NVDAx', name: 'NVIDIA Corp.', price: '180.42', change: '+3.82%', up: true, volume: '$18.4M', tag: 'RWA' },
  { symbol: 'AAPLx', name: 'Apple Inc.', price: '229.16', change: '+1.24%', up: true, volume: '$11.8M', tag: 'RWA' },
  { symbol: 'TSLAX', name: 'Tesla, Inc.', price: '342.78', change: '-0.67%', up: false, volume: '$9.2M', tag: 'RWA' },
  { symbol: 'ETH', name: 'Ether', price: '2,501.90', change: '+2.06%', up: true, volume: '$26.1M', tag: 'CRYPTO' },
  { symbol: 'SPYx', name: 'S&P 500 ETF', price: '641.30', change: '+0.48%', up: true, volume: '$6.7M', tag: 'ETF' }
])

const tabs = ['All', 'Stocks', 'ETFs', 'Crypto']
const filteredMarkets = computed(() => activeTab.value === 'All'
  ? markets.value
  : markets.value.filter(item => activeTab.value === 'Crypto' ? item.tag === 'CRYPTO' : item.tag === (activeTab.value === 'ETFs' ? 'ETF' : 'RWA')))

const shortAddress = computed(() => walletAddress.value ? `${walletAddress.value.slice(0, 6)}...${walletAddress.value.slice(-4)}` : '')

function showToast(message) {
  toast.value = message
  window.clearTimeout(showToast.timeout)
  showToast.timeout = window.setTimeout(() => { toast.value = '' }, 3200)
}

async function connectWallet() {
  walletError.value = ''
  if (!window.ethereum) {
    walletError.value = 'No EVM wallet detected. Install MetaMask or Robinhood Wallet to continue.'
    showToast('EVM wallet not detected')
    return
  }
  try {
    const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' })
    walletAddress.value = accounts?.[0] || ''
    connected.value = Boolean(walletAddress.value)
    if (connected.value) {
      showToast('Wallet connected. Welcome to the signal layer.')
    }
  } catch (error) {
    walletError.value = error?.message || 'Wallet connection was cancelled.'
  }
}

async function addRobinhoodNetwork() {
  if (!window.ethereum) {
    showToast('Connect an EVM wallet first')
    return
  }
  try {
    await window.ethereum.request({
      method: 'wallet_addEthereumChain',
      params: [{
        chainId: '0x1237',
        chainName: 'Robinhood Chain',
        nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
        rpcUrls: ['https://rpc.mainnet.chain.robinhood.com'],
        blockExplorerUrls: ['https://robinhoodchain.blockscout.com']
      }]
    })
    showToast('Robinhood Chain added to your wallet')
  } catch (error) {
    showToast(error?.message || 'Network request was cancelled')
  }
}

function simulateOrder(symbol) {
  if (!connected.value) {
    showToast('Connect your wallet to preview an order')
    return
  }
  showToast(`${symbol} order preview opened. No transaction was sent.`)
}

function scrollTo(id) {
  menuOpen.value = false
  document.getElementById(id)?.scrollIntoView({ behavior: 'smooth' })
}

onMounted(() => {
  timer = window.setInterval(() => { pulse.value = (pulse.value + 1) % 100 }, 2400)
})
onUnmounted(() => window.clearInterval(timer))
</script>

<template>
  <div class="app-shell">
    <div class="ticker-bar" aria-label="Market ticker">
      <div class="ticker-track">
        <span><Radio :size="13" /> LIVE ON ROBINHOOD CHAIN</span>
        <span>NVDAx <b>180.42</b> <i class="up">+3.82%</i></span>
        <span>AAPLx <b>229.16</b> <i class="up">+1.24%</i></span>
        <span>ETH <b>2,501.90</b> <i class="up">+2.06%</i></span>
        <span>24H VOLUME <b>$48.7M</b></span>
        <span class="ticker-muted">SEQUENCER: FIRST-COME, FIRST-SERVED</span>
      </div>
    </div>

    <header class="site-header">
      <a class="brand" href="#top" aria-label="SignalQuiver home">
        <img src="/assets/signalquiver-mark.svg" alt="" />
        <span>SignalQuiver</span>
      </a>
      <nav class="desktop-nav" aria-label="Primary navigation">
        <button @click="scrollTo('terminal')">Terminal</button>
        <button @click="scrollTo('thesis')">Why now</button>
        <button @click="scrollTo('roadmap')">Roadmap</button>
        <a href="https://docs.robinhood.com/chain/" target="_blank" rel="noreferrer">Docs <ExternalLink :size="13" /></a>
      </nav>
      <div class="header-actions">
        <button class="icon-button" aria-label="Open notifications"><Bell :size="17" /><span class="notification-dot"></span></button>
        <button class="wallet-button" :class="{ connected }" @click="connectWallet"><Wallet :size="16" />{{ connected ? shortAddress : 'Connect wallet' }}</button>
        <button class="menu-button" aria-label="Open menu" @click="menuOpen = !menuOpen"><Menu :size="21" /></button>
      </div>
      <div v-if="menuOpen" class="mobile-menu">
        <button @click="scrollTo('terminal')">Terminal <MoveRight :size="16" /></button>
        <button @click="scrollTo('thesis')">Why now <MoveRight :size="16" /></button>
        <button @click="scrollTo('roadmap')">Roadmap <MoveRight :size="16" /></button>
      </div>
    </header>

    <main id="top">
      <section class="hero section-pad">
        <div class="hero-copy">
          <div class="eyebrow"><span class="eyebrow-dot"></span>THE SIGNAL LAYER FOR REAL-WORLD ASSETS</div>
          <h1>See the signal.<br /><em>Settle onchain.</em></h1>
          <p class="hero-lede">SignalQuiver turns the market's highest-conviction conversations into verifiable, executable ideas on Robinhood Chain.</p>
          <div class="hero-actions">
            <button class="primary-button" @click="scrollTo('terminal')">Open terminal <MoveRight :size="17" /></button>
            <button class="text-button" @click="scrollTo('thesis')">Read the thesis <ArrowDownRight :size="16" /></button>
          </div>
          <div class="hero-proof"><ShieldCheck :size="15" /><span>Non-custodial by design</span><span class="proof-separator"></span><span>Built for Robinhood Chain</span></div>
        </div>
        <div class="hero-visual" aria-label="SignalQuiver signal visualization">
          <div class="orbit orbit-a"></div><div class="orbit orbit-b"></div><div class="orbit orbit-c"></div>
          <div class="signal-core"><img src="/assets/signalquiver-mark.svg" alt="" /><span>signal<br /><b>quiver</b></span></div>
          <div class="signal-node node-one"><span class="node-pulse"></span><small>social velocity</small><b>+84.6</b></div>
          <div class="signal-node node-two"><span class="node-pulse cyan"></span><small>quote depth</small><b>$2.4M</b></div>
          <div class="signal-node node-three"><span class="node-pulse yellow"></span><small>settlement</small><b>0.7 sec</b></div>
          <div class="visual-caption"><span class="live-dot"></span> SIGNAL GRAPH / 04:20:16 UTC <span>+</span></div>
        </div>
      </section>

      <section class="signal-ribbon"><div class="ribbon-label">WHAT MOVES THE MARKET</div><div class="ribbon-items"><span>01 &nbsp; Earned social signal</span><span>02 &nbsp; Verifiable price feed</span><span>03 &nbsp; One-click settlement</span><span>04 &nbsp; Your keys, your position</span></div></section>

      <section id="terminal" class="terminal-section section-pad">
        <div class="section-heading"><div><div class="eyebrow"><span class="eyebrow-dot"></span>THE TERMINAL</div><h2>A sharper way to move.</h2></div><div class="chain-status"><span class="live-dot"></span><span><b>Robinhood Chain</b><small>Block 8,420,116 · 0.7s finality</small></span><button aria-label="Add Robinhood Chain" @click="addRobinhoodNetwork"><Plus :size="14" /></button></div></div>
        <div class="terminal-grid">
          <div class="market-panel panel-surface">
            <div class="panel-topline"><div class="panel-title"><BarChart3 :size="17" /> Discover markets</div><button class="filter-button">24h <ChevronDown :size="14" /></button></div>
            <div class="tabs" role="tablist"><button v-for="tab in tabs" :key="tab" :class="{ active: activeTab === tab }" @click="activeTab = tab">{{ tab }}</button></div>
            <div class="market-list">
              <button v-for="market in filteredMarkets" :key="market.symbol" class="market-row" @click="simulateOrder(market.symbol)">
                <span class="market-icon" :class="market.tag.toLowerCase()">{{ market.symbol.slice(0, 1) }}</span><span class="market-name"><b>{{ market.symbol }}</b><small>{{ market.name }}</small></span><span class="market-price"><b>${{ market.price }}</b><small>{{ market.volume }} vol</small></span><span class="market-change" :class="{ up: market.up, down: !market.up }"><component :is="market.up ? ArrowUpRight : ArrowDownRight" :size="15" />{{ market.change }}</span><span class="market-tag">{{ market.tag }}</span>
              </button>
            </div>
            <button class="panel-link" @click="showToast('Full market explorer is coming in the private beta')">View all markets <MoveRight :size="15" /></button>
          </div>
          <div class="order-panel panel-surface">
            <div class="panel-topline"><div class="panel-title"><Sparkles :size="17" /> Signal to order</div><span class="preview-badge">PREVIEW</span></div>
            <div class="order-symbol"><span class="market-icon rwa">N</span><div><b>NVDAx / USDG</b><small>Robinhood Chain · spot</small></div><span class="order-price">$180.42 <i class="up">+3.82%</i></span></div>
            <div class="chart"><div class="chart-grid"></div><svg viewBox="0 0 600 180" preserveAspectRatio="none" aria-hidden="true"><defs><linearGradient id="area" x1="0" x2="0" y1="0" y2="1"><stop offset="0" stop-color="#d8ff3e" stop-opacity=".32"/><stop offset="1" stop-color="#d8ff3e" stop-opacity="0"/></linearGradient></defs><path d="M0 145 C34 136 40 153 71 124 S109 128 139 111 S177 129 206 88 S245 106 274 76 S311 95 340 62 S375 90 402 49 S443 81 470 29 S514 52 542 17 S571 28 600 5 L600 180 L0 180 Z" fill="url(#area)"/><path d="M0 145 C34 136 40 153 71 124 S109 128 139 111 S177 129 206 88 S245 106 274 76 S311 95 340 62 S375 90 402 49 S443 81 470 29 S514 52 542 17 S571 28 600 5" fill="none" stroke="#c5e930" stroke-width="3" vector-effect="non-scaling-stroke"/></svg><span class="chart-label label-low">$164.20</span><span class="chart-label label-high">$182.01</span></div>
            <div class="order-metrics"><span><small>24h high</small><b>$182.01</b></span><span><small>24h low</small><b>$164.20</b></span><span><small>signal score</small><b class="score">84.6 <Sparkles :size="13" /></b></span></div>
            <div class="order-input"><label>Spend</label><div><input value="1,000" aria-label="Spend amount" /><button>USDG <ChevronDown :size="14" /></button></div><small>Estimated receive <b>5.542 NVDAx</b></small></div>
            <button class="primary-button full" @click="simulateOrder('NVDAx')">{{ connected ? 'Preview order' : 'Connect to preview' }} <MoveRight :size="17" /></button>
            <p class="order-note"><ShieldCheck :size="14" /> RFQ quote protected · no custody · no blind routing</p>
          </div>
        </div>
      </section>

      <section id="thesis" class="thesis-section section-pad">
        <div class="thesis-intro"><div class="eyebrow"><span class="eyebrow-dot"></span>THE THESIS</div><h2>Markets are social<br /><em>before they are numerical.</em></h2><p>SignalQuiver gives the earliest information a clean path to execution, without asking users to hand over their assets or their edge.</p><button class="text-button" @click="showToast('Thesis memo saved to your reading list')">Save the memo <Check :size="16" /></button></div>
        <div class="thesis-points"><article><span>01</span><h3>Capture the conversation</h3><p>Rank public market chatter by velocity, source quality, and wallet activity. Noise becomes a legible, time-stamped signal.</p></article><article><span>02</span><h3>Verify the price</h3><p>Pair each idea with onchain liquidity, Chainlink feeds, and RFQ depth before it ever becomes an order.</p></article><article><span>03</span><h3>Settle with control</h3><p>Users route trades from their own wallet on Robinhood Chain. SignalQuiver never takes custody, ever.</p></article></div>
      </section>

      <section class="metrics-section section-pad"><div class="metrics-head"><div class="eyebrow"><span class="eyebrow-dot"></span>NETWORK PULSE</div><span>Updated every 2.4 seconds</span></div><div class="metric-grid"><div><small>Tracked signals</small><b>18,420</b><i class="up">+12.8% <ArrowUpRight :size="14" /></i></div><div><small>Execution paths</small><b>42</b><i class="up">+8 new <ArrowUpRight :size="14" /></i></div><div><small>Protected volume</small><b>$48.7M</b><i class="up">+24.1% <ArrowUpRight :size="14" /></i></div><div><small>Average finality</small><b>0.7s</b><i class="cyan-text">Robinhood L2 <Blocks :size="14" /></i></div></div></section>

      <section id="roadmap" class="roadmap-section section-pad"><div class="roadmap-copy"><div class="eyebrow"><span class="eyebrow-dot"></span>THE QUIVER, IN PUBLIC</div><h2>Build the market<br /><em>you can verify.</em></h2><p>We are building the coordination layer for a world where stocks, crypto, and conviction share one open venue.</p><a class="outline-button" href="https://docs.robinhood.com/chain/" target="_blank" rel="noreferrer">Read the docs <ExternalLink :size="15" /></a></div><div class="roadmap-list"><div class="roadmap-item done"><span class="roadmap-mark"><Check :size="14" /></span><div><small>PHASE 01 · LIVE</small><h3>Signal graph</h3><p>Social velocity, source reputation, and tokenized-stock discovery.</p></div></div><div class="roadmap-item current"><span class="roadmap-mark"><span></span></span><div><small>PHASE 02 · BUILDING</small><h3>Protected execution</h3><p>RFQ routing, wallet-native orders, and portable position history.</p></div></div><div class="roadmap-item"><span class="roadmap-mark"></span><div><small>PHASE 03 · NEXT</small><h3>Community quivers</h3><p>Curated baskets, creator attribution, and permissionless strategies.</p></div></div></div></section>
    </main>

    <footer class="site-footer section-pad"><div class="footer-brand"><a class="brand" href="#top"><img src="/assets/signalquiver-mark.svg" alt="" /><span>SignalQuiver</span></a><p>Signal to settlement for the open market.</p></div><div class="footer-links"><div><small>PRODUCT</small><button @click="scrollTo('terminal')">Terminal</button><button @click="scrollTo('thesis')">Thesis</button><button @click="scrollTo('roadmap')">Roadmap</button></div><div><small>NETWORK</small><a href="https://docs.robinhood.com/chain/" target="_blank" rel="noreferrer">Robinhood Chain <ExternalLink :size="12" /></a><a href="https://robinhoodchain.blockscout.com" target="_blank" rel="noreferrer">Blockscout <ExternalLink :size="12" /></a><button @click="addRobinhoodNetwork">Add network</button></div><div><small>FOLLOW</small><a href="https://x.com/signalquivers" target="_blank" rel="noreferrer">X / @signalquivers <ExternalLink :size="12" /></a><a href="mailto:hello@signalquiver.top">Email us <ExternalLink :size="12" /></a><button @click="showToast('You are on the early access list')">Join early access</button></div></div><div class="footer-bottom"><span>© 2026 SignalQuiver Labs</span><span>Not financial advice. RWA access depends on eligibility and jurisdiction.</span><span class="footer-chain"><span class="live-dot"></span> Robinhood Chain</span></div></footer>

    <div v-if="walletError" class="error-toast"><CircleHelp :size="17" /><span>{{ walletError }}</span><button @click="walletError = ''" aria-label="Close error"><X :size="16" /></button></div>
    <div v-if="toast" class="toast"><Check :size="16" />{{ toast }}</div>
  </div>
</template>
