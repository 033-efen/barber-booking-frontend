
<script>
  let search = ''
  let activeFilter = 'Semua'

  const barbershops = [
    {
      id: 1,
      name: 'Classic Barbershop',
      address: 'Pamekasan Kota',
      rating: '4.9',
      distance: '1.2 km',
      status: 'Buka',
      price: 'Rp25.000',
      image: 'https://images.unsplash.com/photo-1503951914875-452162b0f3f1?w=800'
    },
    {
      id: 2,
      name: 'Gentleman Cut',
      address: 'Jl. Jokotole, Pamekasan',
      rating: '4.8',
      distance: '2.1 km',
      status: 'Buka',
      price: 'Rp30.000',
      image: 'https://images.unsplash.com/photo- barber-placeholder'
    },
    {
      id: 3,
      name: 'Royal Barbershop',
      address: 'Pamekasan, Madura',
      rating: '4.7',
      distance: '3 km',
      status: 'Tutup',
      price: 'Rp20.000',
      image: 'https://images.unsplash.com/photo-1621605815971-fbc98d665033?w=800'
    }
  ]

  let filteredShops = $derived(
    barbershops.filter((shop) => {
      const matchSearch = shop.name.toLowerCase().includes(search.toLowerCase())
      const matchFilter =
        activeFilter === 'Semua' || shop.status === activeFilter

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
      {#each filteredShops as shop}
        <article class="shop-card">
          <div class="shop-image">
            <img src={shop.image} alt={shop.name} />
            <span class:closed={shop.status === 'Tutup'} class="status">
              ● {shop.status}
            </span>
          </div>

          <div class="shop-info">
            <div class="shop-title">
              <h3>{shop.name}</h3>
              <span>★ {shop.rating}</span>
            </div>

            <p class="address">⌖ {shop.address}</p>
            <p class="distance">Jarak: {shop.distance}</p>

            <div class="shop-footer">
              <div>
                <small>Mulai dari</small>
                <strong>{shop.price}</strong>
              </div>
              <button disabled={shop.status === 'Tutup'}>
                Pilih →
              </button>
            </div>
          </div>
        </article>
      {:else}
        <p class="empty">Barbershop tidak ditemukan.</p>
      {/each}
    </div>
  </section>

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
