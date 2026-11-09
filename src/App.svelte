
<script>
  import { onMount, tick } from 'svelte'

  // AUTENTIKASI
  let showAuthModal = $state(false)
  let modeAuth = $state('login')
  let authLoading = $state(false)
  let authError = $state('')
  let authSuccess = $state('')

  let namaAuth = $state('')
  let usernameAuth = $state('')
  let emailAuth = $state('')
  let passwordAuth = $state('')

  let token = $state(
    typeof localStorage !== 'undefined'
      ? localStorage.getItem('token') || ''
      : ''
  )

  let user = $state(
    typeof localStorage !== 'undefined'
      ? JSON.parse(localStorage.getItem('user') || 'null')
      : null
  )

  // RIWAYAT BOOKING
  let riwayatBooking = $state([])
  let riwayatLoading = $state(false)
  let riwayatError = $state('')

    // DATA BOOKING ADMIN
  let semuaBooking = $state([])
  let adminBookingLoading = $state(false)
  let adminBookingError = $state('')
  let adminBookingSuccess = $state('')

  // DATA BARBERSHOP
  let search = $state('')
  let activeFilter = $state('Semua')
  let barbershops = $state([])
  let loading = $state(true)
  let errorMessage = $state('')

  let selectedShop = $state(null)
  let barbers = $state([])
  let services = $state([])
  let detailLoading = $state(false)
  let detailError = $state('')

  // DATA BOOKING
  let selectedBarberId = $state('')
  let selectedServiceId = $state('')
  let namaPelanggan = $state('')
  let tanggalBooking = $state('')
  let jamBooking = $state('')

  let bookingLoading = $state(false)
  let bookingError = $state('')
  let bookingSuccess = $state('')

  // PROSES LOGIN DAN REGISTRASI
  async function prosesAuth() {
    authError = ''
    authSuccess = ''

    if (modeAuth === 'login') {
      if (!usernameAuth.trim() || !passwordAuth) {
        authError = 'Username dan password wajib diisi.'
        return
      }
    } else {
      if (!namaAuth.trim() || !emailAuth.trim() || !passwordAuth) {
        authError = 'Nama, email, dan password wajib diisi.'
        return
      }
    }

    authLoading = true

    try {
      const endpoint = modeAuth === 'login'
        ? '/api/auth/login'
        : '/api/auth/register'

      const body = modeAuth === 'login'
        ? {
            username: usernameAuth.trim(),
            password: passwordAuth
          }
        :  {
            nama: namaAuth.trim(),
            email: emailAuth.trim(),
            password: passwordAuth
          }

      const response = await fetch(endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(body)
      })

      const hasil = await response.json()

      if (!response.ok) {
        throw new Error(hasil.message || 'Autentikasi gagal.')
      }

      if (modeAuth === 'register') {
        authSuccess = 'Registrasi berhasil. Silakan login.'
        modeAuth = 'login'
        namaAuth = ''
        emailAuth = ''
        passwordAuth = ''
        return
      }

      token = hasil.token
      user = hasil.data

      localStorage.setItem('token', token)
      localStorage.setItem('user', JSON.stringify(user))

      showAuthModal = false
      authSuccess = ''
      usernameAuth = ''
      passwordAuth = ''

      if (user.role === 'admin') {
        await muatSemuaBooking()
      } else if (user.role === 'pelanggan') {
        await muatRiwayatBooking()
      }

    } catch (error) {
    authError = error.message
    } finally {
      authLoading = false
    }
  }

  // LOGOUT
   function logout() {
    localStorage.removeItem('token')
    localStorage.removeItem('user')

    token = ''
    user = null

    riwayatBooking = []
    riwayatLoading = false
    riwayatError = ''

    semuaBooking = []
    adminBookingLoading = false
    adminBookingError = ''
    adminBookingSuccess = ''

    authSuccess = ''
    authError = ''

    selectedShop = null
    barbers = []
    services = []
  }

  // MENGAMBIL RIWAYAT BOOKING PELANGGAN
  async function muatRiwayatBooking() {
    if (!token || !user) {
      riwayatBooking = []
      return
    }

    riwayatLoading = true
    riwayatError = ''

    try {
      const response = await fetch('/api/booking/saya', {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${token}`
        }
      })

      const hasil = await response.json()

      if (!response.ok) {
        throw new Error(
          hasil.message || 'Gagal mengambil riwayat booking.'
        )
      }

      riwayatBooking = hasil.data || []

    } catch (error) {
      riwayatError = error.message
    } finally {
      riwayatLoading = false
    }
  }

    // MENGAMBIL SEMUA BOOKING UNTUK ADMIN
  async function muatSemuaBooking() {
    if (!token || !user || user.role !== 'admin') {
      semuaBooking = []
      return
    }

    adminBookingLoading = true
    adminBookingError = ''

    try {
      const response = await fetch('/api/booking', {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${token}`
        }
      })

      const hasil = await response.json()

      if (!response.ok) {
        throw new Error(
          hasil.message || 'Gagal mengambil semua booking.'
        )
      }

      semuaBooking = hasil.data || []

    } catch (error) {
      adminBookingError = error.message
    } finally {
      adminBookingLoading = false
    }
  }

  // FORMAT TANGGAL
  function formatTanggal(tanggal) {
    if (!tanggal) return '-'

    return new Date(tanggal).toLocaleDateString('id-ID', {
      day: '2-digit',
      month: 'long',
      year: 'numeric',
      timeZone: 'Asia/Jakarta'
    })
  }

  // FORMAT HARGA
  function formatHarga(harga) {
    return Number(harga || 0).toLocaleString('id-ID')
  }

  // FORMAT JAM
  function formatJam(jam) {
    return jam ? jam.slice(0, 5) : '-'
  }

  function tanggalHariIni() {
    const sekarang = new Date()
    const tahun = sekarang.getFullYear()
    const bulan = String(sekarang.getMonth() + 1).padStart(2, '0')
    const tanggal = String(sekarang.getDate()).padStart(2, '0')

    return `${tahun}-${bulan}-${tanggal}`
  }

  // MEMILIH BARBERSHOP
  async function pilihBarbershop(shop) {
    selectedShop = shop
    detailLoading = true
    detailError = ''

    barbers = []
    services = []

    try {
      const [responseBarber, responseLayanan] = await Promise.all([
        fetch('/api/barber'),
        fetch('/api/layanan')
      ])

      if (!responseBarber.ok || !responseLayanan.ok) {
        throw new Error('Gagal mengambil data barber dan layanan')
      }

      const hasilBarber = await responseBarber.json()
      const hasilLayanan = await responseLayanan.json()

      barbers = hasilBarber.data.filter(
        (item) =>
          item.barbershop_id === shop.id &&
          item.status === 'aktif'
      )

      services = hasilLayanan.data.filter(
        (item) => item.barbershop_id === shop.id
      )

    } catch (error) {
      detailError = error.message
    } finally {
      await tick()

      document.getElementById('detail-barbershop')?.scrollIntoView({
        behavior: 'smooth',
        block: 'start'
      })

      detailLoading = false
    }
  }

  // MEMBUAT BOOKING
  async function buatBooking() {
    bookingError = ''
    bookingSuccess = ''

    if (!token || !user) {
      bookingError = 'Silakan login terlebih dahulu sebelum booking.'
      showAuthModal = true
      modeAuth = 'login'
      return
    }

    if (
      !namaPelanggan.trim() ||
      !selectedBarberId ||
      !selectedServiceId ||
      !tanggalBooking ||
      !jamBooking
    ) {
      bookingError = 'Mohon lengkapi semua data booking.'
      return
    }

    if (tanggalBooking < tanggalHariIni()) {
      bookingError = 'Tanggal booking tidak boleh di masa lalu.'
      return
    }

    bookingLoading = true

    try {
      const response = await fetch('/api/booking', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          barber_id: Number(selectedBarberId),
          layanan_id: Number(selectedServiceId),
          nama_pelanggan: namaPelanggan.trim(),
          tanggal: tanggalBooking,
          jam: jamBooking
        })
      })

      const hasil = await response.json()

      if (!response.ok) {
        throw new Error(
          hasil.message || hasil.error || 'Booking gagal dibuat.'
        )
      }

      bookingSuccess =
        hasil.message || 'Booking berhasil dikirim!'

      namaPelanggan = ''
      selectedBarberId = ''
      selectedServiceId = ''
      tanggalBooking = ''
      jamBooking = ''

      // Memperbarui riwayat setelah booking berhasil
      if (user.role === 'admin') {
        await muatSemuaBooking()
      } else if (user.role === 'pelanggan') {
        await muatRiwayatBooking()
      }

    } catch (error) {
      bookingError = error.message
    } finally {
      bookingLoading = false
    }
  }

  function kembaliKeDaftar() {
    selectedShop = null
    barbers = []
    services = []
    detailError = ''
  }

  // MENGHITUNG STATUS BUKA
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

    return waktuSekarang >= mulai || waktuSekarang < selesai
  }

  // MENGAMBIL DATA BARBERSHOP
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
    // MENGUBAH STATUS BOOKING OLEH ADMIN
  async function ubahStatusBooking(id, status) {
    adminBookingError = ''
    adminBookingSuccess = ''

    if (status !== 'dikonfirmasi' && status !== 'dibatalkan') {
      adminBookingError = 'Status booking tidak valid.'
      return
    }

    const pesan = status === 'dikonfirmasi'
      ? 'Apakah kamu yakin ingin menerima booking ini?'
      : 'Apakah kamu yakin ingin menolak booking ini?'

    if (!window.confirm(pesan)) {
      return
    }

    try {
      const response = await fetch(`/api/booking/${id}/status`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({ status })
      })

      const hasil = await response.json()

      if (!response.ok) {
        throw new Error(
          hasil.message || 'Gagal mengubah status booking.'
        )
      }

      adminBookingSuccess = status === 'dikonfirmasi'
        ? 'Booking berhasil dikonfirmasi.'
        : 'Booking berhasil ditolak.'

      await muatSemuaBooking()

    } catch (error) {
      adminBookingError = error.message
    }
  }

  // SAAT HALAMAN DIBUKA
  onMount(() => {
    muatBarbershop()

    if (token && user) {
      if (user.role === 'admin') {
        muatSemuaBooking()
      } else if (user.role === 'pelanggan') {
        muatRiwayatBooking()
      }
    }
  })

  // PENCARIAN DAN FILTER
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

    {#if user && user.role === 'pelanggan'}
      <a href="#riwayat">Riwayat Booking</a>
    {/if}
  </nav>

  {#if user}
    <div class="user-menu">
      <span>Halo, {user.nama}</span>
      <button class="login-button" onclick={logout}>
        Keluar
      </button>
    </div>
  {:else}
    <button
      class="login-button"
      onclick={() => {
        modeAuth = 'login'
        showAuthModal = true
        authError = ''
        authSuccess = ''
      }}
    >
      Masuk
    </button>
  {/if}
</header>

<main>
  {#if user && user.role === 'admin'}
    <!-- DASHBOARD ADMIN -->
    <section class="shops-section">
      <div class="section-heading">
        <div>
          <span class="eyebrow">ADMIN PANEL</span>
          <h2>Dashboard Admin</h2>
          <p>Kelola dan pantau seluruh booking pelanggan BARBERBOOK.</p>
        </div>

        <button
          class="back-button"
          onclick={muatSemuaBooking}
          disabled={adminBookingLoading}
        >
          {adminBookingLoading ? 'Memuat...' : 'Refresh Booking'}
        </button>
      </div>

      <div class="shop-grid">
        <article class="shop-card">
          <div class="shop-info">
            <small>Total Booking</small>
            <h3>{semuaBooking.length}</h3>
          </div>
        </article>

        <article class="shop-card">
          <div class="shop-info">
            <small>Menunggu Konfirmasi</small>
            <h3>{semuaBooking.filter(b => b.status === 'menunggu').length}</h3>
          </div>
        </article>

        <article class="shop-card">
          <div class="shop-info">
            <small>Dikonfirmasi</small>
            <h3>{semuaBooking.filter(b => b.status === 'dikonfirmasi').length}</h3>
          </div>
        </article>
      </div>

      {#if adminBookingSuccess}
        <p class="booking-success">{adminBookingSuccess}</p>
      {/if}

      {#if adminBookingError}
        <div class="empty">
          <p>{adminBookingError}</p>
          <button onclick={muatSemuaBooking}>Coba Lagi</button>
        </div>
      {:else if adminBookingLoading}
        <p class="empty">Sedang memuat semua booking...</p>
      {:else if semuaBooking.length > 0}
        <div class="shop-grid">
          {#each semuaBooking as booking (booking.id)}
            <article class="shop-card">
              <div class="shop-info">
                <h3>Booking #{booking.id}</h3>

                <p><strong>Pelanggan:</strong> {booking.nama_pelanggan}</p>
                <p><strong>Barber:</strong> {booking.nama_barber}</p>
                <p><strong>Layanan:</strong> {booking.nama_layanan}</p>
                <p><strong>Harga:</strong> {formatHarga(booking.harga)}</p>
                <p><strong>Tanggal:</strong> {formatTanggal(booking.tanggal)}</p>
                <p><strong>Jam:</strong> {formatJam(booking.jam)}</p>
                <p>
                  <strong>Status:</strong>
                  <span class="status">{booking.status}</span>
                </p>

                {#if booking.status === 'menunggu'}
                  <div class="shop-footer">
                    <button
                      class="booking-submit"
                      onclick={() => ubahStatusBooking(booking.id, 'dikonfirmasi')}
                    >
                      Terima
                    </button>

                    <button
                      class="back-button"
                      onclick={() => ubahStatusBooking(booking.id, 'dibatalkan')}
                    >
                      Tolak
                    </button>
                  </div>
                {/if}
              </div>
            </article>
          {/each}
        </div>
      {:else}
        <div class="empty">
          <p>Belum ada booking dari pelanggan.</p>
        </div>
      {/if}
    </section>
  {:else}
    <!-- HALAMAN PELANGGAN -->
  <section class="hero" id="home">
    <div class="hero-content">
      <span class="eyebrow">✦ BOOK YOUR STYLE</span>

      <h1>
        Temukan Gaya Terbaikmu,
        <span>Mulai dari Sini.</span>
      </h1>

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

  <!-- DAFTAR BARBERSHOP -->
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

  <!-- DETAIL BARBERSHOP -->
  {#if selectedShop}
    <section
      id="detail-barbershop"
      class="shops-section detail-section"
    >
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
          <p class="address">Alamat: {selectedShop.alamat}</p>
          <p>Telepon: {selectedShop.telepon || 'Belum tersedia'}</p>

          <p>
            Jam operasional:
            {selectedShop.jam_buka} - {selectedShop.jam_tutup}
          </p>

          <p>
            Status:
            <strong>{selectedShop.statusBuka ? 'Buka' : 'Tutup'}</strong>
          </p>
        </div>
      </div>

      {#if detailLoading}
        <p class="empty">Memuat data barber dan layanan...</p>

      {:else if detailError}
        <p class="empty">{detailError}</p>

      {:else}
        <div class="detail-list">
          <h3>Barber Kami</h3>

          <div class="detail-grid">
            {#each barbers as barber (barber.id)}
              <article class="shop-card detail-item">
                <div class="shop-info">
                  <h4>{barber.nama}</h4>
                  <p>Barber profesional</p>
                  <span class="status">● Aktif</span>
                </div>
              </article>
            {:else}
              <p>Belum ada barber tersedia.</p>
            {/each}
          </div>
        </div>

        <div class="detail-list">
          <h3>Layanan dan Harga</h3>

          <div class="detail-grid">
            {#each services as service (service.id)}
              <article class="shop-card detail-item">
                <div class="shop-info">
                  <h4>{service.nama_layanan}</h4>

                  <p class="service-price">
                    Rp{Number(service.harga).toLocaleString('id-ID')}
                  </p>

                  <p>Durasi: {service.durasi} menit</p>
                </div>
              </article>
            {:else}
              <p>Belum ada layanan tersedia.</p>
            {/each}
          </div>
        </div>
      {/if}
    </section>
  {/if}

  <!-- FORM BOOKING -->
  <div class="detail-list booking-section">
    <h3>Booking Jadwal</h3>
    <p>Isi data berikut untuk memesan jadwal potong rambut.</p>

    <div class="booking-form">
      <label>
        Nama Pelanggan
        <input
          type="text"
          bind:value={namaPelanggan}
          placeholder="Masukkan nama kamu"
        />
      </label>

      <label>
        Pilih Barber
        <select bind:value={selectedBarberId}>
          <option value="">Pilih barber</option>

          {#each barbers as barber}
            <option value={String(barber.id)}>
              {barber.nama}
            </option>
          {/each}
        </select>
      </label>

      <label>
        Pilih Layanan
        <select bind:value={selectedServiceId}>
          <option value="">Pilih layanan</option>

          {#each services as service}
            <option value={String(service.id)}>
              {service.nama_layanan} -
              Rp{Number(service.harga).toLocaleString('id-ID')}
            </option>
          {/each}
        </select>
      </label>

      <label>
        Tanggal Booking
        <input
          type="date"
          bind:value={tanggalBooking}
          min={tanggalHariIni()}
        />
      </label>

      <label>
        Jam Booking
        <input
          type="time"
          bind:value={jamBooking}
        />
      </label>

      {#if bookingError}
        <p class="booking-error">{bookingError}</p>
      {/if}

      {#if bookingSuccess}
        <p class="booking-success">{bookingSuccess}</p>
      {/if}

      <button
        class="booking-submit"
        onclick={buatBooking}
        disabled={bookingLoading}
      >
        {bookingLoading ? 'Mengirim...' : 'Konfirmasi Booking'}
      </button>
    </div>
  </div>

  <!-- DETAIL BARBERSHOP KEDUA (DIPERTAHANKAN DARI KODE LAMA) -->
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

  <!-- RIWAYAT BOOKING PELANGGAN -->
  {#if user && user.role === 'pelanggan'}
    <section class="shops-section" id="riwayat">
      <div class="section-heading">
        <div>
          <span class="eyebrow">MY BOOKING</span>
          <h2>Riwayat Booking Saya</h2>
          <p>Lihat daftar pemesanan yang pernah kamu lakukan.</p>
        </div>

        <button
          class="back-button"
          onclick={muatRiwayatBooking}
          disabled={riwayatLoading}
        >
          {riwayatLoading ? 'Memuat...' : '↻ Refresh'}
        </button>
      </div>

      {#if riwayatLoading}
        <p class="empty">Sedang memuat riwayat booking...</p>

      {:else if riwayatError}
        <div class="empty">
          <p>{riwayatError}</p>

          <button onclick={muatRiwayatBooking}>
            Coba Lagi
          </button>
        </div>

      {:else if riwayatBooking.length > 0}
        <div class="shop-grid">
          {#each riwayatBooking as booking (booking.id)}
            <article class="shop-card">
              <div class="shop-info">
                <div class="shop-title">
                  <h3>Booking #{booking.id}</h3>

                  <span class="status">
                    {booking.status}
                  </span>
                </div>

                <p class="address">
                  Barber: {booking.nama_barber || '-'}
                </p>

                <p>
                  Layanan: {booking.nama_layanan || '-'}
                </p>

                <p>
                  Harga: Rp{formatHarga(booking.harga)}
                </p>

                <p>
                  Durasi: {booking.durasi || '-'} menit
                </p>

                <p>
                  Tanggal: {formatTanggal(booking.tanggal)}
                </p>

                <p>
                  Jam: {formatJam(booking.jam)}
                </p>
              </div>
            </article>
          {/each}
        </div>

      {:else}
        <div class="empty">
          <p>Kamu belum memiliki riwayat booking.</p>

          <a href="#barbershops" class="primary-button">
            Cari Barbershop
          </a>
        </div>
      {/if}
    </section>
  {/if}

  <!-- TENTANG -->
  <section class="about-section" id="about">
    <span class="eyebrow">WHY BARBERBOOK?</span>

    <h2>
      Booking Lebih Mudah, Potong Rambut Lebih Nyaman.
    </h2>

    <p>
      Satu platform untuk menemukan barbershop, memilih barber,
      dan mengatur jadwal potong rambut.
    </p>
  </section>

  {/if}

  <!-- MODAL LOGIN DAN REGISTRASI -->
  {#if showAuthModal}
    <div class="auth-overlay">
    
      <div class="auth-modal">
        <button
          class="auth-close"
          onclick={() => showAuthModal = false}
        >
          ×
        </button>

        <h2>
          {modeAuth === 'login' ? 'Selamat Datang!' : 'Buat Akun Baru'}
        </h2>

        <p>
          {modeAuth === 'login'
            ? 'Silakan masuk ke akun Anda.'
            : 'Daftar untuk mulai melakukan booking.'}
        </p>

        <form
          onsubmit={(e) => {
            e.preventDefault()
            prosesAuth()
          }}
        >
        {#if modeAuth === 'register'}
          <label for="namaAuth">Nama Lengkap</label>

          <input
            id="namaAuth"
            type="text"
            bind:value={namaAuth}
            placeholder="Masukkan nama lengkap"
            required
          />
        {/if}

        {#if modeAuth === 'login'}
          <label for="usernameAuth">Username</label>

          <input
            id="usernameAuth"
            type="text"
            bind:value={usernameAuth}
            placeholder="Masukkan username"
            required
          />
        {:else}
          <label for="emailAuth">Email</label>

          <input
            id="emailAuth"
            type="email"
            bind:value={emailAuth}
            placeholder="Masukkan email"
            required
          />
        {/if}

        <label for="passwordAuth">Password</label>

        <input
          id="passwordAuth"
          type="password"
          bind:value={passwordAuth}
          placeholder="Masukkan password"
          required
          minlength="6"
        />

          {#if authError}
            <p class="auth-error">{authError}</p>
          {/if}

          {#if authSuccess}
            <p class="auth-success">{authSuccess}</p>
          {/if}

          <button
            type="submit"
            class="auth-submit"
            disabled={authLoading}
          >
            {authLoading
              ? 'Memproses...'
              : modeAuth === 'login' ? 'Masuk' : 'Daftar'}
          </button>
        </form>

        <div class="auth-switch">
          {#if modeAuth === 'login'}
            Belum punya akun?

            <button
              onclick={() => {
                modeAuth = 'register'
                authError = ''
                authSuccess = ''
              }}
            >
              Daftar sekarang
            </button>
          {:else}
            Sudah punya akun?

            <button
              onclick={() => {
                modeAuth = 'login'
                authError = ''
                authSuccess = ''
              }}
            >
              Masuk
            </button>
          {/if}
        </div>
      </div>
    </div>
  {/if}
</main>

<footer>
  <a href="/" class="brand">
    <span class="brand-icon">B</span>
    BARBER<span>BOOK</span>
  </a>

  <p>
    © 2026 BARBERBOOK. Booking barbershop jadi lebih mudah.
  </p>
</footer>
