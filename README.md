# STM32_UART_GPIO_Control
//stm32 için C kodu. STM32 için IDE indirdim ama anlayıp çözüp orda donanım kurana kadar vaktim yetmedi. Sadece PC13 ve P14'ü output olarak ayarlayabilmeyi başardım.
/* USER CODE BEGIN PV */
// PV: Private Variables (Özel Değişkenler)

// UART üzerinden gelen 2 karakterlik komutu saklamak için bir tampon (buffer).
uint8_t rx_buffer[2];

// UART kesmesi tarafından alınan tek bir karakteri geçici olarak tutar.
uint8_t rx_char;

// Gelen komutun kaçıncı karakterinde olduğumuzu sayan bir indeks. (0 veya 1)
uint8_t rx_index = 0;

/* USER CODE END PV */

/* USER CODE BEGIN 0 */

/**
  * @brief  Gelen 2 karakterlik komutu işleyen fonksiyon.
  * @note   Bu fonksiyon '0', '1' veya '2' karakterlerini alıp
  * ilgili GPIO pinlerini SET, RESET veya TOGGLE yapar.
  * @retval None
  */
void process_command(void)
{
  // --- BİRİNCİ PİN KONTROLÜ (rx_buffer[0]'a göre) ---

  if (rx_buffer[0] == '0')
  {
    // Eğer ilk karakter '0' ise, LED1 pinini SÖNDÜR (LOW).
    HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_RESET);
  }
  else if (rx_buffer[0] == '1')
  {
    // Eğer ilk karakter '1' ise, LED1 pinini YAK (HIGH).
    HAL_GPIO_WritePin(LED1_GPIO_Port, LED1_Pin, GPIO_PIN_SET);
  }
  else if (rx_buffer[0] == '2')
  {
    // Eğer ilk karakter '2' ise, LED1 pininin durumunu TERSİNE ÇEVİR (TOGGLE).
    HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
  }
  // (Geçersiz karakterler göz ardı edilir)

  // --- İKİNCİ PİN KONTROLÜ (rx_buffer[1]'e göre) ---

  if (rx_buffer[1] == '0')
  {
    // Eğer ikinci karakter '0' ise, LED2 pinini SÖNDÜR (LOW).
    HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_RESET);
  }
  else if (rx_buffer[1] == '1')
  {
    // Eğer ikinci karakter '1' ise, LED2 pinini YAK (HIGH).
    HAL_GPIO_WritePin(LED2_GPIO_Port, LED2_Pin, GPIO_PIN_SET);
  }
  else if (rx_buffer[1] == '2')
  {
    // Eğer ikinci karakter '2' ise, LED2 pininin durumunu TERSİNE ÇEVİR (TOGGLE).
    HAL_GPIO_TogglePin(LED2_GPIO_Port, LED2_Pin);
  }
  // (Geçersiz karakterler göz ardı edilir)
}


/**
  * @brief  UART Veri Alımı Tamamlandığında çağrılan Geri Bildirim (Callback) fonksiyonu.
  * @note   Bu fonksiyon, bir 'interrupt' (kesme) tarafından tetiklenir.
  * Her bir karakter alındığında otomatik olarak çalışır.
  * @param  huart: Kesmeyi tetikleyen UART donanımı.
  * @retval None
  */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
  // Sadece USART1'den gelen kesmeleri işleme al
  if (huart->Instance == USART1)
  {
    // 1. Gelen karakteri (rx_char) ana tamponumuza (rx_buffer) kaydet.
    rx_buffer[rx_index] = rx_char;

    // 2. Bir sonraki karakter için indeksi artır.
    rx_index++;

    // 3. İki karakter de alındı mı?
    if (rx_index == 2)
    {
      // Evet, 2 karakter de alındı.
      // İndeksi bir sonraki komut için sıfırla.
      rx_index = 0;

      // Alınan komutu işlemek için fonksiyonumuzu çağır.
      process_command();
    }

    // 4. Kritik Adım: UART kesmesini bir sonraki karakteri alması için yeniden etkinleştir.
    //    Bu olmazsa, sistem sadece bir karakter alır ve durur.
    HAL_UART_Receive_IT(&huart1, &rx_char, 1);
  }
}

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init(); // HAL kütüphanesini ve temel sistem donanımlarını başlatır.

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config(); // .ioc dosyasında ayarlanan saat yapılandırmasını (72MHz) yükler.

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init(); // GPIO pinlerini (PC13, PC14) çıkış olarak ayarlar.
  MX_USART1_UART_Init(); // USART1'i (9600 baud) ayarlar.
  
  /* USER CODE BEGIN 2 */
  
  // --- BAŞLANGIÇ ---
  // UART kesmesini ilk kez başlatıyoruz.
  // Sisteme: "USART1 üzerinden 1 adet karakter al, bittiğinde
  //          'HAL_UART_RxCpltCallback' fonksiyonunu tetikle ve
  //          bu karakteri 'rx_char' değişkenine kaydet." diyoruz.
  HAL_UART_Receive_IT(&huart1, &rx_char, 1);

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    // ANA DÖNGÜ BOŞ.
    // Tüm işler CPU'yu kilitlemeden, arka planda 'kesmeler' (interrupts)
    // tarafından yönetiliyor. Bu, verimli gömülü sistem tasarımının
    // temelidir. CPU, bu sırada başka görevleri (eğer olsaydı)
    // yapabilirdi.

    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
