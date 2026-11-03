
<script>
  import { onMount } from 'svelte'

  let search = $state('')
  let activeFilter = $state('Semua')
  let barbershops = $state([])
  let loading = $state(true)
  let errorMessage = $state('')

  let selectedShop = $state(null)

  function pilihBarbershop(shop) {
    selectedShop = shop
  }

  function kembaliKeDaftar() {
    selectedShop = null
    }
    
  // Menghitung status buka berdasarkan jam operasional
  function cekStatusBuka(jamBuka, jamTutup) {
    if (!jamBuka || !jamTutup) return false

    const sekarang = new Date()
    const waktuSekarang =
      sekarang.getHours() * 3600 +
      sekarang.getMinutes() * 60 +
      sekarang.getSeconds()

    const [jam1, menit1, detik1] = jamBuka.split(':').map(Number)
    const [jam2, menit2, detik2] = jamTutup.split(':').map(Number)

    const mulai = jam1 * 3600 + menit1 * 60 + (detik1 || 0)
    const selesai = jam2 * 3600 + menit2 * 60 + (detik2 || 0)

    if (mulai <= selesai) {
      return waktuSekarang >= mulai && waktuSekarang < selesai
    }

    // Mendukung barbershop yang buka melewati tengah malam
    return waktuSekarang >= mulai || waktuSekarang < selesai
  }

  // Mengambil data dari backend
  async function muatBarbershop() {
    loading = true
    errorMessage = ''

    try {
      const response = await fetch('/api/barbershop')

      if (!response.ok) {
        throw new Error('Gagal mengambil data barbershop')
      }

      const hasil = await response.json()

      barbershops = hasil.data
        .filter((shop) => shop.status === 'aktif')
        .map((shop) => ({
          ...shop,
          statusBuka: cekStatusBuka(shop.jam_buka, shop.jam_tutup)
        }))
    } catch (error) {
      errorMessage = error.message
    } finally {
      loading = false
    }
  }

  onMount(() => {
    muatBarbershop()
  })

  // Pencarian dan filter
  let filteredShops = $derived(
    barbershops.filter((shop) => {
      const matchSearch =
        shop.nama.toLowerCase().includes(search.toLowerCase()) ||
        shop.alamat.toLowerCase().includes(search.toLowerCase())

      const status = shop.statusBuka ? 'Buka' : 'Tutup'

      const matchFilter =
        activeFilter === 'Semua' || status === activeFilter

      return matchSearch && matchFilter
    })
  )
</script>

<svelte:head>
  <title>BARBERBOOK - Booking Barbershop Pamekasan</title>
  <meta
    name="description"
    content="Temukan dan booking barbershop favoritmu di Pamekasan."
  />
</svelte:head>

<header class="navbar">
  <a href="/" class="brand">
    <span class="brand-icon">B</span>
    BARBER<span>BOOK</span>
  </a>

  <nav>
    <a href="#home">Beranda</a>
    <a href="#barbershops">Barbershop</a>
    <a href="#about">Tentang</a>
  </nav>

  <button class="login-button">Masuk</button>
</header>

<main>
  <section class="hero" id="home">
    <div class="hero-content">
      <span class="eyebrow">✦ BOOK YOUR STYLE</span>
      <h1>Temukan Gaya Terbaikmu, <span>Mulai dari Sini.</span></h1>
      <p>
        Cari barbershop terbaik di Pamekasan, pilih barber favoritmu,
        dan booking jadwal potong rambut tanpa harus antre.
      </p>

      <a href="#barbershops" class="primary-button">
        Cari Barbershop <span>→</span>
      </a>

      <div class="hero-stats">
        <div><strong>100%</strong><small>Praktis</small></div>
        <div><strong>24/7</strong><small>Akses Booking</small></div>
        <div><strong>1 Tempat</strong><small>Banyak Pilihan</small></div>
      </div>
    </div>

    <div class="hero-image">
      <img
        src="https://images.unsplash.com/photo-1622286342621-4bd786c2447c?w=1000"
        alt="Suasana barbershop modern"
      />
      <div class="image-label">
        <span>✦</span>
        Your Style, Your Choice
      </div>
    </div>
  </section>

  <section class="shops-section" id="barbershops">
    <div class="section-heading">
      <div>
        <span class="eyebrow">FIND YOUR BARBER</span>
        <h2>Barbershop Pilihan</h2>
        <p>Temukan tempat potong rambut di sekitar Pamekasan.</p>
      </div>
      <span class="location-label">⌖ Pamekasan, Madura</span>
    </div>

    <div class="search-panel">
      <div class="search-box">
        <span>⌕</span>
        <input
          bind:value={search}
          placeholder="Cari nama barbershop..."
        />
      </div>

      <div class="filters">
        {#each ['Semua', 'Buka', 'Tutup'] as filter}
          <button
            class:active={activeFilter === filter}
            onclick={() => activeFilter = filter}
          >
            {filter}
          </button>
        {/each}
      </div>
    </div>


<div class="shop-grid">
  {#if loading}
    <p class="empty">Sedang memuat data barbershop...</p>

  {:else if errorMessage}
    <div class="empty">
      <p>{errorMessage}</p>
      <button onclick={muatBarbershop}>Coba Lagi</button>
    </div>

  {:else if filteredShops.length > 0}
    {#each filteredShops as shop (shop.id)}
      <article class="shop-card">
        <div class="shop-image">
          <img
            src="https://images.unsplash.com/photo-1503951914875-452162b0f3f1?w=800"
            alt={shop.nama}
          />

          <span
            class:closed={!shop.statusBuka}
            class="status"
          >
            ● {shop.statusBuka ? 'Buka' : 'Tutup'}
          </span>
        </div>

        <div class="shop-info">
          <div class="shop-title">
            <h3>{shop.nama}</h3>
          </div>

          <p class="address">⌖ {shop.alamat}</p>

          <p class="distance">
            Jam operasional: {shop.jam_buka || '--:--'} -
            {shop.jam_tutup || '--:--'}
          </p>

          <div class="shop-footer">
            <div>
              <small>Telepon</small>
              <strong>{shop.telepon || 'Belum tersedia'}</strong>
            </div>

            <button onclick={() => pilihBarbershop(shop)}>
              Pilih →
            </button>

          </div>
        </div>
      </article>
    {/each}

  {:else}
    <p class="empty">Barbershop tidak ditemukan.</p>
  {/if}
</div>

  </section>
  {#if selectedShop}
    <section class="shops-section detail-section">
      <button class="back-button" onclick={kembaliKeDaftar}>
        ← Kembali ke daftar
      </button>

      <div class="section-heading">
        <div>
          <span class="eyebrow">BARBERSHOP DETAIL</span>
          <h2>{selectedShop.nama}</h2>
          <p>Informasi lengkap barbershop pilihanmu.</p>
        </div>
      </div>

      <div class="shop-card detail-card">
        <div class="shop-info">
          <h3>Informasi Barbershop</h3>

          <p class="address">
            Alamat: {selectedShop.alamat}
          </p>

          <p>
            Telepon: {selectedShop.telepon || 'Belum tersedia'}
          </p>

          <p>
            Jam operasional:
            {selectedShop.jam_buka} - {selectedShop.jam_tutup}
          </p>

          <p>
            Status:
            <strong>
              {selectedShop.statusBuka ? 'Buka' : 'Tutup'}
            </strong>
          </p>
        </div>
      </div>
    </section>
  {/if}
  
  <section class="about-section" id="about">
    <span class="eyebrow">WHY BARBERBOOK?</span>
    <h2>Booking Lebih Mudah, Potong Rambut Lebih Nyaman.</h2>
    <p>
      Satu platform untuk menemukan barbershop, memilih barber,
      dan mengatur jadwal potong rambut.
    </p>
  </section>
</main>

<footer>
  <a href="/" class="brand">
    <span class="brand-icon">B</span>
    BARBER<span>BOOK</span>
  </a>
  <p>© 2026 BARBERBOOK. Booking barbershop jadi lebih mudah.</p>
</footer>
