<template>
  <ion-page>
    <ion-header :translucent="true" class="custom-header">
      <ion-toolbar class="custom-toolbar">
        <ion-title> 💰 DATA CRYPTOCURRENCY</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="fetchCryptoData">Refresh</ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div id="container">
        <div v-if="isLoading" class="status-message">
          <p>Memuat data...</p>
        </div>

        <div v-else-if="error" class="status-message error-message">
          <p>Gagar memuat data: {{ error }}</p>
        </div>

        <ion-list v-else-if="cryptoData.length" class="crypto-list">
          <ion-item-divider>
            <ion-label>Rank | Name</ion-label>
            <ion-label slot="end" class="header-end">Symbol | USD</ion-label>
          </ion-item-divider>

          <ion-item v-for="item in cryptoData" :key="item.id">
            <ion-label>
              <div class="rank-name-container">
                <span class="rank-box">{{ item.rank }}</span>
                <span class="coin-name">{{ item.name }}</span>
              </div>
            </ion-label>

            <ion-label slot="end" class="price-symbol-container">
              <span class="symbol-text">{{ item.symbol }}</span>
              <span class="price-text">${{ item.price_usd }}</span>
            </ion-label>
          </ion-item>
        </ion-list>

        <div v-else class="status-message">
          <p>Belum ada data yang tersedia.</p>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonList,
  IonItem,
  IonLabel,
  IonItemDivider,
  IonButtons,
  IonButton,
} from "@ionic/vue";

// --- DEFINISI TYPE (INTERFACE) ---
// Sama seperti di kode Node.js sebelumnya, tapi di sini kita hanya perlu type data
interface Ticker {
  id: string;
  symbol: string;
  name: string;
  rank: number;
  price_usd: string;
}

interface ApiResponse {
  data: Ticker[];
  info: any; // Tidak terlalu penting untuk tampilan
}

// --- STATE REACTIVE ---
const API_URL = "https://api.coinlore.net/api/tickers/";
const cryptoData = ref<Ticker[]>([]); // Array untuk menyimpan data yang ditampilkan
const isLoading = ref(true); // Status loading
const error = ref<string | null>(null); // Status error

// --- FUNGSI PENGAMBIL DATA (CORE LOGIC) ---
async function fetchCryptoData() {
  isLoading.value = true;
  error.value = null; // Reset error

  try {
    const response = await axios.get<ApiResponse>(API_URL);
    const tickers = response.data.data;

    // Filter & Format Data untuk Tampilan (mirip dengan tugas sebelumnya)
    const formattedData: Ticker[] = tickers
      .filter((t) => t.rank <= 10) // Ambil 10 teratas
      .map((ticker) => ({
        id: ticker.id,
        rank: ticker.rank,
        name: ticker.name,
        symbol: ticker.symbol,
        // Format harga ke 4 desimal
        price_usd: parseFloat(ticker.price_usd).toFixed(4),
      }));

    cryptoData.value = formattedData;
  } catch (err: any) {
    // Penanganan error menggunakan isAxiosError dari Axios (membutuhkan type TS yang benar)
    if (err.response) {
      error.value = `Gagal Koneksi (${err.response.status || "No Status"})`;
      console.error("Axios Error:", err.message);
    } else {
      error.value = "Terjadi kesalahan koneksi jaringan atau tak terduga.";
      console.error("Unknown Error:", err);
    }
    cryptoData.value = [];
  } finally {
    isLoading.value = false;
  }
}

// --- LIFECYCLE HOOKS ---
// Panggil fungsi pengambilan data saat komponen selesai dimuat (mount)
onMounted(() => {
  fetchCryptoData();
});
</script>

<style scoped>
/* Pengaturan Kontainer */
#container {
  padding: 30px;
  position: static; /* Ganti dari absolute agar konten mengalir normal */
  transform: none;
  text-align: left;
}

/* Penyesuaian List & Item */
ion-list {
  background: none;
  padding: 0;
}

ion-item {
  --padding-start: 10px;
  --inner-padding-end: 10px;
  /* Tambahkan garis pemisah di bawah setiap item */
  --border-width: 0 0 1px 0;
  --border-color: #000000; /* Warna gelap untuk tema gelap */
}

ion-item-divider {
  font-weight: bold;
  color: #ffffff;
}

/* Kontainer utama untuk Rank dan Name */
.rank-name-container {
  display: flex;
  align-items: center;
  gap: 10px; /* Jarak antara Rank dan Nama */
}

/* Styling Kotak Rank (sesuai mockup) */
.rank-box {
  background: rgb(125, 125, 17); /* Warna biru untuk kotak rank */
  color: rgb(255, 253, 253);
  padding: 4px 8px;
  border-radius: 4px;
  font-weight: bold;
  min-width: 30px;
  text-align: center;
}

.coin-name {
  font-weight: bold;
  font-size: 1.1em;
}

/* Kontainer untuk Symbol dan Price (Kolom Kanan) */
.price-symbol-container {
  display: flex;
  flex-direction: column; /* Susun Symbol di atas Price */
  align-items: flex-end; /* Ratakan konten ke kanan */
}

.symbol-text {
  font-weight: bold;
  font-size: 1.1em;
  color: var(--ion-color-primary);
}

.price-text {
  font-size: 0.95em;
  color: var(--ion-color-success, #2dd36f);
  font-weight: 500;
}

/* Penyesuaian Header Divider */
.header-end {
  text-align: right;
  min-width: 150px;
}

.status-message {
  padding: 20px;
  text-align: center;
}

.error-message {
  color: var(--ion-color-danger);
}

.custom-toolbar {
  --background: rgb(125, 125, 17) !important;
  --color: rgb(255, 255, 255);
  font-weight: bold;
}

.custom-header {
  /* Menambahkan padding 30px di seluruh sisi header */
  padding: 40px;
  box-shadow: none;
}
</style>
