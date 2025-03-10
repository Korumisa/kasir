#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <time.h>

// Makro untuk menentukan jumlah produk dan panjang maksimal nama produk
#define MAX_PRODUCTS 5         // Jumlah produk yang tersedia
#define MAX_NAME_LENGTH 25     // Panjang maksimal nama produk

/* ------------------------------------------------------------------
   Struktur data untuk produk
   - Mengelompokkan informasi nama dan harga produk.
------------------------------------------------------------------ */
typedef struct {
    char name[MAX_NAME_LENGTH]; // Nama produk
    int price;                  // Harga produk
} Product;

/* ------------------------------------------------------------------
   Struktur data untuk transaksi (order)
   - Menyimpan informasi transaksi pembelian: nama produk, harga,
     jumlah, total harga, dan diskon.
------------------------------------------------------------------ */
typedef struct {
    char name[MAX_NAME_LENGTH]; // Nama produk yang dibeli
    int price;                  // Harga produk
    int quantity;               // Jumlah produk yang dibeli
    float total;                // Total harga (price * quantity)
    float discount;             // Diskon yang diberikan
} Transaction;

/* ------------------------------------------------------------------
   Deklarasi array produk
   - Data produk yang tersedia dalam aplikasi.
------------------------------------------------------------------ */
Product products[MAX_PRODUCTS] = {
    {"Laptop",      5000000},
    {"Smartphone",  3000000},
    {"Headset",     600000},
    {"Mouse",       150000},
    {"Keyboard",    450000}
};

/* ------------------------------------------------------------------
   Deklarasi array transaksi
   - Menyimpan transaksi yang dilakukan oleh pengguna.
   - Variabel numTransactions menyimpan jumlah transaksi saat ini.
------------------------------------------------------------------ */
Transaction transactions[MAX_PRODUCTS];
int numTransactions = 0; // Jumlah transaksi yang sudah dilakukan

/* ------------------------------------------------------------------
   Fungsi calculateDiscount
   - Menghitung diskon berdasarkan jumlah pembelian:
     * Diskon 15% jika pembelian > 5 item
     * Diskon 10% jika pembelian > 3 item
   - Parameter: price (harga produk), quantity (jumlah yang dibeli)
   - Mengembalikan nilai diskon (float)
------------------------------------------------------------------ */
float calculateDiscount(int price, int quantity) {
    if (quantity > 5)
        return price * quantity * 0.15f;
    else if (quantity > 3)
        return price * quantity * 0.10f;
    return 0;
}

/* ------------------------------------------------------------------
   Fungsi resetTransaction
   - Mengosongkan keranjang belanja dengan mereset jumlah transaksi
     dan mengatur semua elemen array transactions menjadi 0.
------------------------------------------------------------------ */
void resetTransaction() {
    numTransactions = 0;
    memset(transactions, 0, sizeof(transactions));
}

/* ------------------------------------------------------------------
   Fungsi getCurrentTime
   - Mengembalikan waktu saat ini dalam bentuk string.
   - Menggunakan fungsi time, localtime, dan asctime.
------------------------------------------------------------------ */
char *getCurrentTime() {
    time_t t = time(NULL);
    return asctime(localtime(&t));
}

/* ------------------------------------------------------------------
   Fungsi generateUniqueID
   - Menghasilkan ID unik untuk struk berdasarkan waktu saat ini.
   - Mengalokasikan memori untuk ID dan mengembalikannya sebagai string.
------------------------------------------------------------------ */
char *generateUniqueID() {
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    char *id = malloc(20);
    sprintf(id, "%d%d%d%d%d%d",
            tm.tm_year + 1900, tm.tm_mon + 1,
            tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec);
    return id;
}

/* ------------------------------------------------------------------
   Fungsi showCart
   - Menampilkan daftar transaksi (keranjang belanja) yang sudah dimasukkan.
   - Mencetak nomor transaksi, nama produk, dan jumlah pembelian.
------------------------------------------------------------------ */
void showCart() {
    printf("\n=========================================\n");
    printf("         Keranjang Belanja Anda         \n");
    printf("=========================================\n");
    for (int i = 0; i < numTransactions; i++) {
        if (transactions[i].quantity > 0) {
            printf("| %d | %-16s | %d x\n", i + 1, transactions[i].name, transactions[i].quantity);
        }
    }
    printf("=========================================\n");
}

/* ------------------------------------------------------------------
   Fungsi printReceipt
   - Mencetak struk transaksi ke file .txt dengan format:
     Header Toko, informasi struk, daftar pembelian dengan format kolom,
     dan ringkasan transaksi.
   - Parameter:
     * totalCost    : Total harga sebelum diskon.
     * totalDiscount: Total diskon.
     * totalDue     : Tagihan yang harus dibayar.
     * payment      : Jumlah uang yang dibayarkan.
     * change       : Kembalian.
------------------------------------------------------------------ */
void printReceipt(int totalCost, int totalDiscount, int totalDue, int payment, int change) {
    FILE *fptr;

    // Generate ID unik dan buat nama file untuk struk
    char *receiptID = generateUniqueID();
    char filename[strlen(receiptID) + 5];
    sprintf(filename, "%s.txt", receiptID);

    // Buka file untuk menulis struk
    fptr = fopen(filename, "w");
    if (!fptr) {
        printf("Gagal membuat file struk.\n");
        free(receiptID);
        return;
    }

    // Tulis header struk dengan informasi toko
    fprintf(fptr, "|==========================================================|\n");
    fprintf(fptr, "|                         Toko SKENSA                      |\n");
    fprintf(fptr, "|            Jl. HOS Cokroaminoto No. 84, Denpasar         |\n");
    fprintf(fptr, "|                            Bali                          |\n");
    fprintf(fptr, "|                      Telp : 0816285791                   |\n");
    fprintf(fptr, "|                                                          |\n");
    fprintf(fptr, "|ID Struk : %-48s|\n", receiptID);
    fprintf(fptr, "|==========================================================|\n");
    fprintf(fptr, "| Nama Barang\t  |   Harga   | Total Harga | Diskon\t   |\n");
    fprintf(fptr, "|==========================================================|\n");

    // Tulis detail transaksi (hanya transaksi dengan quantity > 0)
    for (int i = 0; i < MAX_PRODUCTS; i++) {
        if (transactions[i].quantity > 0) {
            fprintf(fptr, "| %dx %-13s| Rp %-6d | Rp %-9.0f | Rp %-6.0f\t   |\n",
                    transactions[i].quantity,
                    transactions[i].name,
                    transactions[i].price,
                    transactions[i].total,
                    transactions[i].discount);
        }
    }
    fprintf(fptr, "|==========================================================|\n");
    fprintf(fptr, "|                                                          |\n");
    fprintf(fptr, "|Total Harga  = Rp %-33d|\n", totalCost);
    fprintf(fptr, "|Total Diskon = Rp %-33d|\n", totalDiscount);
    fprintf(fptr, "|Tagihan      = Rp %-33d|\n", totalDue);
    fprintf(fptr, "|Pembayaran   = Rp %-33d|\n", payment);
    fprintf(fptr, "|Kembalian    = Rp %-33d|\n", change);
    fprintf(fptr, "|                                                          |\n");
    fprintf(fptr, "|                                                          |\n");
    fprintf(fptr, "|                                                          |\n");

    // Tulis waktu pencetakan struk
    char *currentTime = getCurrentTime();
    fprintf(fptr, "|Waktu : %-55s|\n", currentTime);
    fprintf(fptr, "|==========================================================|\n");

    fclose(fptr);
    free(receiptID);
    printf("\nStruk telah disimpan dengan nama file: %s\n", filename);
}

/* ------------------------------------------------------------------
   Fungsi main
   - Menampilkan menu produk dan menangani input pengguna.
   - Proses meliputi penambahan transaksi, reset transaksi,
     proses pembayaran dan pencetakan struk, serta validasi pembayaran.
   - Program berjalan sampai pengguna memilih untuk keluar.
------------------------------------------------------------------ */
int main() {
    int choice = 0, quantity = 0, totalCost = 0, totalDiscount = 0;
    int totalDue = 0, payment = 0, change = 0;
    bool running = true;

    // Tampilkan header dan menu awal aplikasi dengan format baru
    printf("Selamat datang di Toko SKENSA\n");
    printf("Silahkan pilih barang yang Anda inginkan\n\n");
    printf("===================================\n");
    printf("| No |\t   Barang    |    Harga   |\n");
    printf("===================================\n");
    for (int i = 0; i < MAX_PRODUCTS; i++) {
        printf("| %-2d | %-13s |  Rp %-7d|\n", i + 1, products[i].name, products[i].price);
    }
    printf("===================================\n\n");
    printf("99. Struk Pembayaran\n");
    printf("55. Reset Pilihan\n");
    printf("00. Keluar\n\n");
    printf("===================================\n");

    // Mulai loop utama untuk menerima input pengguna
    while (running) {
        printf("\nInput pilihan yang Anda inginkan: ");
        scanf("%d", &choice);

        if (choice == 99) {
            // Proses pembayaran dan pencetakan struk
            if (numTransactions > 0) {
                // Hitung total harga dan total diskon
                for (int i = 0; i < numTransactions; i++) {
                    totalCost     += (int)transactions[i].total;
                    totalDiscount += (int)transactions[i].discount;
                }
                totalDue = totalCost - totalDiscount;

                // Tampilkan ringkasan transaksi sebelum pembayaran
                printf("\n=======================================================\n");
                printf("| No | Jumlah | Barang          | Harga    | Total    | Diskon  |\n");
                printf("=======================================================\n");
                for (int i = 0; i < numTransactions; i++) {
                    printf("| %-2d | %-6d | %-16s | Rp %-6d | Rp %-6.0f | Rp %-6.0f|\n",
                           i + 1,
                           transactions[i].quantity,
                           transactions[i].name,
                           transactions[i].price,
                           transactions[i].total,
                           transactions[i].discount);
                }
                printf("=======================================================\n");
                printf("Total Harga  : Rp %d\n", totalCost);
                printf("Total Diskon : Rp %d\n", totalDiscount);
                printf("Total Bayar  : Rp %d\n", totalDue);
                printf("=======================================================\n");

                // Validasi pembayaran: ulangi input jika uang yang dibayar kurang
                do {
                    printf("Masukkan uang pembayaran (Rp): ");
                    scanf("%d", &payment);
                    if (payment < totalDue) {
                        printf("\n[ERROR] Maaf, uang Anda masih kurang.\n");
                        printf("Silakan masukkan nominal yang cukup untuk membayar total tagihan.\n\n");
                    }
                } while (payment < totalDue);

                change = payment - totalDue;
                printf("\nPembayaran berhasil!\n");
                printf("Kembalian Anda: Rp %d\n", change);

                // Cetak struk transaksi ke file .txt
                printReceipt(totalCost, totalDiscount, totalDue, payment, change);

                printf("\nTerima kasih telah berbelanja di Toko Skensa!\n");
                running = false;
            } else {
                printf("Anda belum memiliki pesanan!\n");
            }
        }
        else if (choice == 0) {
            running = false;
        }
        else if (choice == 55) {
            resetTransaction();
            totalCost = totalDiscount = totalDue = payment = change = 0;
            printf("\nPilihan Anda telah di-reset.\n");
        }
        else if (choice < 1 || choice > MAX_PRODUCTS) {
            printf("Pilihan tidak sesuai!\n");
        }
        else {
            printf("Masukkan jumlah pembelian: ");
            scanf("%d", &quantity);

            transactions[numTransactions].quantity = quantity;
            strcpy(transactions[numTransactions].name, products[choice - 1].name);
            transactions[numTransactions].price = products[choice - 1].price;
            transactions[numTransactions].total = (float)(products[choice - 1].price * quantity);
            transactions[numTransactions].discount = calculateDiscount(products[choice - 1].price, quantity);
            numTransactions++;

            // Tampilkan keranjang belanja yang telah diperbarui
            showCart();
        }
    }
    return 0;
}
