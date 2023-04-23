# ESP32-Sleep-Mode
## Sleep mode là gì?
Trên thực tế, tất cả các dòng vi điều khiển đều có chế độ ngủ (sleep mode) nhưng hiệu suất của mỗi dòng sẽ khác nhau. Ngày nay các thiết bị IOT dạng wearable rất phổ biến và cần sử dụng pin để đảm bảo tính linh hoạt.  Sleep Mode ra đời để giúp MCU tiết kiệm năng lượng bằng cách tắt các phần chưa cần thiết và đặt các sự kiện có thể đánh thức (Wake Up) MCU.
## Các chế độ tiết kiệm năng lượng của ESP32 Sleep Mode
Trong ESP32 có 5 chế độ năng lượng đó là:
- Active mode
- Modem Sleep mode
- Light Sleep mode
- Deep Sleep mode
- Hibernation mode
### ESP32 Active Mode
Trong chế độ này, tất cả các  khối chức năng của chip đều được bật, và đây là chế độ bình thường khi chạy các chương trình
Vì Active Mode (hay còn gọi là Full Power Mode) bật mọi chức năng (đặc biệt là module WiFi, Bluetooth và nhân xử lý ), nên chip yêu cầu dòng điện hơn 240mA để hoạt động. Ngoài ra, nếu bạn sử dụng cả hai chức năng WiFi và Bluetooth cùng lúc thì năng lượng tiêu thụ sẽ lớn hơn rất nhiều (lớn nhất là 790mA).
Lưu ý: Vì thế nên khi sử dụng các KIT esp32 chúng ta cần cấp nguồn ngoài cho KIT nếu không wifi hoặc BT/BLE sẽ không hoạt động nếu phải điều khiển thêm nhiều các phần tử khác trong mạch
### ESP32 Modem Sleep
Trong Modem Sleep, mọi thứ đều hoạt động, chỉ có WiFi, Bluetooth và Radio bị tắt. Để duy trì sự hoạt động của Wifi và BT/BLE, chúng phải được đánh thức định kì theo một thời gian mà lập trình viên mong muốn. Chế độ này chỉ hoạt động trong chế độ máy trạm (Wifi Station), không được sử dụng trong chế độ điểm truy cập (Access Point). Khi ở chế độ này ESP32 vẫn kết nối với bộ định tuyến thông qua cơ chế DTIM Beacon. Để tiết kiệm năng lượng, ESP32 vô hiệu hóa module Wi-Fi giữa hai khoảng thời gian DTIM Beacon và tự động thức dậy trước khi đến Beacon tiếp theo. Thời gian wake up có thể từ 100 – 1000ms.
Với chế độ này ESP32 tiêu thụ từ: 3mA – 20mA
### ESP32 Light Sleep
Với chế độ này ESP32 tắt các thành phần phát sóng như wifi, BT/BLE như chế độ modem sleep, cùng với tăt nguồn xung đồng hồ (Clock) của CPU và RAM và ngoại vi. Nhưng RTC và ULP – coprocessor vẫn hoạt động bình thường. Kĩ thuật này được gọi là clock-gating. Clock gating là một kỹ thuật để giảm tiêu thụ năng lượng. Nó vô hiệu hóa các phần của mạch điện bằng cách tắt các xung clock, để các flip-flop trong chúng không phải chuyển trạng thái. Vì khi có chuyển đổi trạng thái thì năng lượng bị tiêu thụ, ngược lại, mức tiêu thụ năng lượng bằng 0. Trong chế độ này ESP32 tiêu thụ khoảng 0.8mA
### ESP32 Deep Sleep
Ở chế độ Deep Sleep, CPU, RAM và tất cả các ngoại vi đều bị tắt. Các bộ phận duy nhất của chip vẫn được cấp nguồn là: bộ RTC, ngoại vi RTC (bao gồm bộ ULP) và bộ nhớ RTC. CPU chính bị tắt nguồn còn bộ ULP thực hiện các phép đo cảm biến và đánh thức hệ thống chính dựa trên dữ liệu đo được. Cùng với CPU, bộ nhớ chính của chip cũng bị tắt. Vì vậy, mọi thứ được lưu trữ trong bộ nhớ đó bị xóa sạch và không thể truy cập được. Tuy nhiên, bộ nhớ RTC vẫn được bật. Vì vậy, nội dung của nó được bảo quản trong Deep Sleep và có thể được lấy ra sau khi chip được đánh thức. Đó là lý do mà chip lưu trữ dữ liệu kết nối Wi-Fi và Bluetooth trong bộ nhớ RTC trước đó. Khi Wake up khỏi Deep Sleep, ESP32 sẽ hoạt động lại từ đầu, tương tự như việc reset vậy. Trong chế độ này ESP32 tiêu thụ từ 10µA đến 0.15mA 
### ESP32 Hibernation
Trong chế độ Hibernation Mode, chip vô hiệu hóa bộ tạo dao động 8MHz bên trong và bộ ULP. Bộ nhớ phục hồi RTC cũng bị tắt nguồn, có nghĩa là không có cách nào chúng ta có thể lưu trữ dữ liệu trong chế độ ngủ đông. Mọi thứ khác đều bị tắt ngoại trừ bộ đếm thời gian RTC slow clock và một số GPIO RTC đang hoạt động. Chúng có trách nhiệm đánh thức chip ra khỏi Hibernation Mode. Vậy nên hãy lưu ý khi sử dụng chế độ này nhé. Trong chế độ Hibernation Mode, chip chỉ tiêu thụ khoảng 2.5µA.

## Các nguồn dùng để đánh thức ESP32 khỏi chế độ Sleep
### Timer Wakeup
Bộ RTC có một timer tích hợp có thể được sử dụng để đánh thức chip sau một khoảng thời gian đã được xác định trước. Thời gian được chỉ định với độ chính xác micro giây, nhưng độ phân giải thực tế phụ thuộc vào nguồn xung đã chọn cho RTC SLOW_CLK. 
Chế độ đánh thức này không yêu cầu các thiết bị ngoại vi hoặc bộ nhớ RTC được bật nguồn trong khi ngủ. Hàm esp_sleep_enable_timer_wakeup()có thể được sử dụng để kích hoạt chế độ đánh thức ngủ sâu bằng cách sử dụng một timer.
#### Ứng dụng Timer trong Light Sleep
Đầu tiên, ta khai báo các thư viện cần thiết, trong đó có "esp_sleep.h" là thư viện chứa các nguyên mẫu hàm phục vụ cho các chế độ Sleep. 
Để chọn timer làm nguồn đánh thức ESP32 khỏi chế độ sleep, ta sử dụng hàm esp_sleep_enable_timer_wakeup. Hàm này có nguyên mẫu là 
esp_err_t esp_sleep_enable_timer_wakeup(uint64_t time_in_us)
Trong đó: 
time_in_us: tham số này dùng để chỉ thời gian ta muốn ESP32 ở trạng thái Sleep, đơn vị thời gian là micro giây (us)
Ví dụ: Muốn ESP32 ở trong chế độ sleep trong 5 giây, ta sẽ gán tham số time_in_us = 5000000
Tiếp theo, để ESP32 bước vào chế độ light sleep, chúng ta gọi hàm esp_light_sleep_start();
Trước khi ESP32 bước vào chế độ light sleep, chúng ta sẽ gọi hàm print để đưa ra một tin nhắn thông báo 
Ngoài ra, để kiểm chứng thời gian ESP32 ở trong chế độ Light Sleep, chúng ta sẽ gọi hàm esp_timer_get_time để tiến hành đo thời gian tại lúc bắt đầu và kết thúc thời gian ESP32 ở trong chế độ Light Sleep. Bên dưới là chi tiết phần code tham khảo chương trình ứng dụng 
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include "esp_sleep.h"
#include "esp_log.h"
#include "esp32/rom/uart.h"
#include "esp_timer.h"

void app_main()
{
    esp_sleep_enable_timer_wakeup(5000000);
    printf("going for a nap\n");
    uart_tx_wait_idle(CONFIG_ESP_CONSOLE_UART_NUM);
    int64_t before = esp_timer_get_time();
    esp_light_sleep_start();
    int64_t after = esp_timer_get_time();
    printf("napped for %lld\n", (after - before) / 1000);
}
```
Khi thực hiện chương trình, ESP32 sẽ kích hoạt timer làm nguồn wake up sau đó gửi một tin nhắn thông báo trước khi bước vào chế độ Light  Sleep. Sau đó sử dụng esp_timer_get_time để tính toán thời gian ở trong chế độ Light Sleep. Cụ thể, tôi đã cài đặt thời gian để ESP32 bước vào chế độ Light Sleep trong 5 giây.
#### Ứng dụng Timer trong Deep Sleep
Tương tự với Light Sleep, chúng ta cũng sẽ sử dụng hàm esp_sleep_enable_timer_wakeup để chọn Timer là nguồn đánh thức ESP32 khỏi chế độ Deep Sleep.
Sự khác biệt giữa Light Sleep và Deep Sleep ở phần wake up đó là wake up ở Light sleep thì ESP32 sẽ tiếp tục những công việc đang dang dở trước đó, còn ở Deep sleep thì khi wake up, ESP32 sẽ khởi động lại (reset), các dữ liệu trước đó đều mất.
Để ESP32 bước vào chế độ light sleep, chúng ta gọi hàm esp_deep_sleep_start()
Ngoài ra, Với ESP32, ta có thể lưu dữ liệu trên bộ nhớ RTC. ESP32 có 8kB SRAM trên phần RTC, được gọi là RTC fast memory. Dữ liệu được lưu ở đây không bị xóa khi ESP vào chế độ Deep Sleep. Tuy nhiên, nó sẽ bị xóa khi nhấn nút Reset (nút có nhãn EN trên bo mạch ESP32).
Để lưu dữ liệu trên bộ nhớ RTC, ta chỉ cần thêm RTC_DATA_ATTR trước khi định nghĩa biến, và biến này phải ở trạng thái global. Ví dụ lưu biến timesWokenUp trên bộ nhớ RTC. Biến này sẽ đếm số lần ESP32 đã thức dậy sau khi Deep Sleep. Bên dưới là chi tiết phần code tham khảo chương trình ứng dụng 
```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_sleep.h"

RTC_DATA_ATTR int timesWokenUp = 0;
void app_main(void)
{
  esp_sleep_enable_timer_wakeup(5 * 1000000);
  printf("going to sleep. woken up %d\n", timesWokenUp++);

  esp_deep_sleep_start();
}
```
Khi thực hiện chương trình. ESP32 sẽ kích hoạt timer làm nguồn wake up sau đó đưa ra một dòng thông báo ESP32 bước vào chế độ deep sleep và số lần được đánh thức 
### Touch-pad Wakeup
Touchpad wakeup hoặc touch wakeup là tùy chọn khác để đánh thức bo mạch ESP32 từ chế độ ngủ sâu. Việc đánh thức sẽ xảy ra khi người dùng chạm vào một trong các chân cảm ứng của bo mạch ESP32 gây ra một ngắt cảm ứng.  Hàm esp_sleep_enable_touchpad_wakeup()được sử dụng để kích hoạt đánh thức từ chế độ ngủ sâu thông qua touchpad. 
ESP-WROOM-32 bao gồm 10 cảm biến touch trên bo mạch. Chúng hữu ích vì chúng hoạt động như các cảm biến cảm ứng có thể gây ra đánh thức ngắt touchpad khi chúng được chạm vào, phát hiện bất kỳ sóng điện/magnet xung quanh chúng. Các chân cảm biến cảm ứng được trang bị trên bo mạch ESP32:
•	TOUCH0 – GPIO4
•	TOUCH1 – GPIO0
•	TOUCH2 – GPIO2
•	TOUCH3 – GPIO15
•	TOUCH4 – GPIO13
•	TOUCH5 – GPIO12
•	TOUCH6 – GPIO14
•	TOUCH7 – GPIO27
•	TOUCH8 – GPIO33
•	TOUCH9 – GPIO32

### External Wakeup (ext0) 
Bên cạnh đó, các nguồn đánh thức bên ngoài cũng thường được sử dụng, trong đó sự thay đổi trạng thái của chân GPIO sẽ đánh thức bo mạch ESP32 từ chế độ Deep Sleep. Nguồn đánh thức được cấu hình trước khi đặt bo mạch ESP32 vào chế độ Deep Sleep. Có hai loại ngắt đánh thức bên ngoài mà chúng ta có thể thiết lập: ext0 và ext1. Trong ext0, một chân GPIO được cấu hình để hoạt động như một nguồn đánh thức bên ngoài. Tuy nhiên, nếu ta muốn sử dụng nhiều chân GPIO, thì ext1 sẽ được sử dụng. Một điểm quan trọng cần lưu ý, ta chỉ có thể sử dụng các chân GPIO RTC để đánh thức bên ngoài. ESP32 DevKit V1-DOIT có 14 chân GPIO RTC có thể được sử dụng để gọi đánh thức ngắt bên ngoài: 
•	RTC_GPIO0 : GPIO36
•	RTC_GPIO3 : GPIO39
•	RTC_GPIO9 : GPIO32
•	RTC_GPIO8 : GPIO33
•	RTC_GPIO6 : GPIO25
•	RTC_GPIO7 : GPIO26
•	RTC_GPIO17: GPIO27
•	RTC_GPIO16: GPIO14
•	RTC_GPIO15: GPIO12
•	RTC_GPIO14: GPIO13
•	RTC_GPIO11: GPIO0
•	RTC_GPIO13: GPIO15
•	RTC_GPIO12: GPIO2
•	RTC_GPIO10: GPIO4
Mô-đun RTC IO chứa các logic để kích hoạt đánh thức khi một trong các chân RTC GPIO được đặt thành một mức logic được xác định trước. RTC IO là một phần của miền nguồn điện ngoại vi RTC, vì vậy các thiết bị ngoại vi RTC sẽ được giữ nguồn trong khi ngủ sâu nếu nguồn đánh thức này được yêu cầu.
Bởi vì mô-đun RTC IO được kích hoạt trong chế độ này, các điện trở kéo lên hoặc kéo xuống nội bộ cũng có thể được sử dụng. Chúng cần được cấu hình bởi ứng dụng sử dụng các hàm rtc_gpio_pullup_en() và rtc_gpio_pulldown_en() trước khi gọi esp_sleep_start().
Trong các phiên bản 0 và 1 của ESP32, nguồn đánh thức này không tương thích với các nguồn đánh thức ULP và touch.
Hàm esp_sleep_enable_ext0_wakeup()có thể được sử dụng để kích hoạt nguồn đánh thức này. 
Nguyên mẫu của hàm như sau:
esp_err_t esp_sleep_enable_ext0_wakeup(gpio_num_t gpio_num, int level);
Trong đó:                                                                                                                                                           gpio_num: là tên của chân GPIO ta chọn làm nguồn cho ext0  
level: là mức logic của chân GPIO ta chọn  
Sau khi đánh thức từ chế độ ngủ, chân RTC IO được dùng để đánh thức sẽ được cấu hình lại chân GPIO thông thường bằng cách sử dụng hàm rtc_gpio_deinit(gpio_num).
#### Ứng dụng ext0 trong Deep Sleep
Đầu tiên chúng ta sử dụng một nút nhấn, kết nối nút nhấn này với một chân GPIO trên esp32 để làm một external interrupt. Khi nhấn nút, mức logic thay đổi, điều này sẽ kích hoạt đánh thức esp32 khỏi chế độ sleep. Để sử dụng được chân GPIO trong chế độ Deep Sleep, chúng ta cần sử dụng header file "driver/rtc_io.h". Header file này chứa các hàm chức năng cho phép ta sử dụng và cấu hình hoạt động của các chân GPIO trong chế độ Deep Sleep. Trong ví dụ này, nhóm em sẽ sử dụng chân GPIO 0. Vì ở trạng thái thông thường, các chân GPIO của Esp32 luôn ở trạng thái trở kháng cao (High-impedance) hoặc thả nổi (Floating) rất khó để xác định mức logic để đưa vào tham số trong hàm esp_sleep_enable_ext0_wakeup.  Vì thế, chân GPIO này sẽ được cấu hình pull-up để trạng thái của chân GPIO 0 luôn ở mức logic 1. Hàm esp_sleep_enable_ext0_wakeup được dùng để chọn external interrupt 0 làm nguồn đánh thức ESP32. Sau đó ESP32 được đưa vào chế độ Deep Sleep bằng hàm esp_deep_sleep_start(). Bên dưới là chi tiết phần code tham khảo chương trình ứng dụng 
```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_sleep.h"
#include "driver/rtc_io.h"

#define BUTTON GPIO_NUM_0

RTC_DATA_ATTR int timesWokenUp = 0;
void app_main(void)
{
  rtc_gpio_deinit(BUTTON);
  /// general gpio functions using the pin.

  rtc_gpio_pullup_en(BUTTON);
  rtc_gpio_pulldown_dis(BUTTON);
  esp_sleep_enable_ext0_wakeup(BUTTON,0);
  printf("going to sleep. woken up %d\n", timesWokenUp++);
  esp_deep_sleep_start();
}
```
Khi chạy chương trình ứng dụng. ESP32 sẽ được đánh thức mỗi khi ta nhấn nút tương ứng với chân GPIO 0 nằm trên board mạch.
### External Wakeup (ext1) 
Như đã đề cập trước đó, ext1 được sử dụng khi nhiều chân GPIO RTC được sử dụng để hoạt động như nguồn đánh thức bên ngoài. Để kích hoạt nguồn báo thức ext1, chúng ta sử dụng API sau: esp_sleep_enable_ext1_wakeup
Nguyên mẫu của hàm như sau:
esp_err_t esp_sleep_enable_ext1_wakeup(uint64_t gpio_pin_mask, esp_sleep_ext1_wakeup_mode_t mode);
Trong đó:
gpio_pin_mask:  Mặt nạ bit của số GPIO sẽ đánh thức. Chỉ những chân GPIO có chức năng RTC có thể được sử dụng.
mode: chọn chức năng logic được sử dụng để xác định điều kiện đánh thức:
•	ESP_EXT1_WAKEUP_ALL_LOW: Trong trường hợp của chức năng logic này, báo thức được kích hoạt khi tất cả các chân RTC GPIO được đặt đều ở trạng thái LOW .
•	ESP_EXT1_WAKEUP_ANY_HIGH: Trong trường hợp của chức năng logic này, báo thức được kích hoạt khi bất kỳ chân RTC GPIO nào được đặt là ở trạng thái HIGH.
Bộ điều khiển RTC chịu trách nhiệm kích hoạt báo thức ext1. Do đó, các thiết bị ngoại vi RTC có thể được tắt nguồn trong khi sử dụng nguồn báo thức ext1. Để kích hoạt một báo thức bằng cách sử dụng ext1, chúng ta có thể sử dụng một trong hai chức năng logic: ESP_EXT1_WAKEUP_ALL_LOW hoặc ESP_EXT1_WAKEUP_ANY_HIGH.
Nguồn thức dậy này được thực hiện bởi bộ điều khiển RTC. Vì vậy, các bộ phận và bộ nhớ RTC có thể được tắt nguồn ở chế độ này. Tuy nhiên, nếu các bộ phận RTC bị tắt nguồn, các trở kéo lên và kéo xuống bên trong GPIO sẽ bị vô hiệu hóa. Để sử dụng trở kéo lên hoặc kéo xuống bên trong, yêu cầu ngoại vi RTC được cấp nguồn trong khi ngủ, bằng cách sử dụng hàm esp_sleep_pd_config và cấu hình trở kéo lên/kéo xuống bằng các chức năng rtc_gpio_, trước khi vào chế độ ngủ:
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_ON);
gpio_pullup_dis(gpio_num);
gpio_pulldown_en(gpio_num);
Sau khi thức dậy khỏi chế độ ngủ, (các) chân GPIO RTC được sử dụng để đánh thức sẽ được định cấu hình trở về các chân GPIO thông thường bằng hàm rtc_gpio_deinit()
#### Ứng dụng ext1 trong Deep Sleep
Tương tự với ext0, để sử dụng được chân GPIO trong chế độ Deep Sleep, chúng ta cần sử dụng header file "driver/rtc_io.h". Trong ví dụ này, nhóm em sẽ sử dụng chân GPIO 25 và chân GPIO 26. Cài đặt cấu hình cho 2 chân GPIO này như sau:
Trường hợp 1: Nếu sử dụng chế độ  ESP_EXT1_WAKEUP_ALL_LOW thì ta sẽ cấu hình tất cả các chân RTC GPIO ở dạng pullup, để các chân GPIO luôn có mức logic ban đầu là 1, khi nhấn nút thì mức logic sẽ về 0 để đúng với điều kiện của ESP_EXT1_WAKEUP_ALL_LOW.
Trường hợp 2: Nếu sử dụng chế độ  ESP_EXT1_WAKEUP_ANY_HIGH thì ta sẽ cấu hình tất cả các chân RTC GPIO ở dạng pulldown, để các chân GPIO có mức logic ban đầu là 0, khi nhấn nút thì mức logic sẽ là 1 để đúng với điều kiện của ESP_EXT1_WAKEUP_ANY_HIGH.
Ngoài ra, chúng ta sẽ phải bật ngoại vi RTC để có thể sử dụng các chân RTC GPIO bằng lệnh esp_sleep_pd_config. Tiếp theo, ta sẽ tạo biến mask để đưa vào làm tham số của hàm esp_sleep_enable_ext1_wakeup.
uint64_t mask = (1ULL << BUTTON_1) | (1ULL << BUTTON_2);
Giá trị của biến mask sẽ là 1ULL (Unsigned Long Long) nếu cả BUTTON_1 và BUTTON_2 đều bằng 0. Bên dưới là chi tiết chương trình ứng dụng EXT1 với Deep Sleep.
```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_sleep.h"
#include "driver/rtc_io.h"

#define BUTTON_1 GPIO_NUM_25
#define BUTTON_2 GPIO_NUM_26

RTC_DATA_ATTR int timesWokenUp = 0;
void app_main(void)
{
  rtc_gpio_deinit(BUTTON_1);
  rtc_gpio_deinit(BUTTON_2);
  /// general gpio functions using the pin.

  esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_ON);
  rtc_gpio_pullup_dis(BUTTON_1);
  rtc_gpio_pulldown_en(BUTTON_1);
  rtc_gpio_pullup_dis(BUTTON_2);
  rtc_gpio_pulldown_en(BUTTON_2);

  uint64_t mask = (1ULL << BUTTON_1) | (1ULL << BUTTON_2);
  esp_sleep_enable_ext1_wakeup(mask, ESP_EXT1_WAKEUP_ANY_HIGH);
  printf("going to sleep. woken up %d\n", timesWokenUp++);

  esp_deep_sleep_start();
}
```

### ULP Coprocessor Wakeup
ESP32 cũng được tích hợp bộ xử lý công suất thấp được gọi là ULP coProcessor (Ultra-Low Power). Điểm đặc biệt của bộ xử lý này là nó có thể chạy độc lập với bộ xử lý lõi chính và nó cũng có quyền truy cập vào một số thiết bị ngoại vi. ULP có thể được sử dụng để kiểm tra các cảm biến, giám sát giá trị ADC hoặc cảm biến cảm ứng và đánh thức MCU khi phát hiện một sự kiện cụ thể. ULP cocoprocessor  là một phần của ngoại vi RTC, và nó thực hiện chương trình được lưu trong bộ nhớ RTC slow memory. RTC slow memory được cấp nguồn trong quá trình ESP32 bước vào chế độ sleep nếu được yêu. Ngoại vi RTC tự động được cấp nguồn trước khi ULP coProcessor bắt đầu chạy chương; Khi dừng chạy, ngoại vi RTC được tự động ngắt nguồn.
Hàm esp_sleep_enable_ulp_wakeup() được sử dụng để kích hoạt nguồn đánh thức là ULP coProcessor.
### GPIO wakeup ( chỉ dành cho light sleep) 
Ngoài các nguồn đánh thức EXT0 và EXT1 được mô tả ở trên, một phương pháp đánh thức khác từ các đầu vào bên ngoài có sẵn trong chế độ Light Sleep. Với nguồn đánh thức này, từng chân GPIO có thể được cấu hình độc lập để kích hoạt đánh thức trên mức cao hoặc thấp bằng cách sử dụng chức năng gpio_wakeup_enable(). Khác với các nguồn đánh thức EXT0 và EXT1, chỉ có thể được sử dụng với các RTC IO, nguồn đánh thức này có thể được sử dụng với bất kỳ loại chân GPIO nào (RTC hoặc digital).
Hàm esp_sleep_enable_gpio_wakeup() có thể được sử dụng để kích hoạt nguồn đánh thức này.
Trước khi vào chế độ Light Sleep, ta phải kiểm tra xem bất kỳ chân GPIO nào được điều khiển có phần nguồn VDD_SDIO hay không. Nếu có, nguồn này phải được cấu hình để vẫn hoạt động trong khi ngủ.
Ví dụ, trên bo mạch ESP32-WROOM-32, GPIO16 và GPIO17 được liên kết với lĩnh vực nguồn VDD_SDIO. Nếu chúng được cấu hình để giữ mức cao trong khi ngủ nhẹ, nguồn này sẽ được cấu hình để tiếp tục được cấp nguồn. Điều này có thể được thực hiện bằng cách sử dụng hàm esp_sleep_pd_config():
esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_VDDSDIO, ESP_PD_OPTION_ON);
#### Ứng dụng GPIO trong Light Sleep
Bảng 2-5 là chi tiết phần code ứng dụng. đầu tiên chúng ta cấu hình ngõ vào INPUT_PIN là một ngõ vào GPIO, cho phép đánh thức (wakeup) khi có tín hiệu xuống mức thấp (low level). Tiếp theo, chúng ta bật chế độ đánh thức GPIO và cấu hình đánh thức theo timer với thời gian chờ là 5000000 micro giây (tương đương với 5 giây). 

Sau đó, trong vòng lặp vô hạn, ta kiểm tra trạng thái INPUT_PIN. Nếu nó đang ở mức thấp, chúng ta in ra thông báo yêu cầu người dùng thả nút và đợi đến khi nó được thả ra (thông qua hàm vTaskDelay để delay trong một thời gian nhất định và hàm rtc_gpio_get_level để kiểm tra trạng thái của INPUT_PIN).
Nếu ngõ vào INPUT_PIN đang ở trạng thái cao, chúng ta in ra thông báo và đợi một khoảng thời gian ngắn để UART hoạt động cho đến khi gửi xong tất cả các dữ liệu, sau đó chúng ta bắt đầu chế độ sleep nhẹ bằng cách sử dụng hàm esp_light_sleep_start. 
Sau khi kết thúc thời gian sleep, chúng ta sử dụng hàm esp_sleep_get_wakeup_cause để kiểm tra nguyên nhân của wakeup và in ra thông tin về thời gian nghỉ và nguyên nhân của wakeup.
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <sys/time.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_sleep.h"
#include "esp_log.h"
#include "esp32/rom/uart.h"
#include "driver/rtc_io.h"


#define INPUT_PIN 0

void app_main()
{
    gpio_pad_select_gpio(INPUT_PIN);
    gpio_set_direction(INPUT_PIN, GPIO_MODE_INPUT);
    gpio_wakeup_enable(INPUT_PIN, GPIO_INTR_LOW_LEVEL);

    esp_sleep_enable_gpio_wakeup();
    esp_sleep_enable_timer_wakeup(5000000);

    while (true)
    {
        if (rtc_gpio_get_level(INPUT_PIN) == 0)
        {
            printf("please release button\n");
            do
            {
                vTaskDelay(pdMS_TO_TICKS(10));
            } while (rtc_gpio_get_level(INPUT_PIN) == 0);
        }

        printf("going for a nap\n");
        uart_tx_wait_idle(CONFIG_ESP_CONSOLE_UART_NUM);

        int64_t before = esp_timer_get_time();

        esp_light_sleep_start();

        int64_t after = esp_timer_get_time();

        esp_sleep_wakeup_cause_t reason = esp_sleep_get_wakeup_cause();

        printf("napped for %lld, reason was %s\n", (after - before) / 1000,
               reason == ESP_SLEEP_WAKEUP_TIMER ? "timer" : "button");
    }
}
```
### UART wakeup (chỉ dành cho light sleep) 
Khi ESP32 nhận được đầu vào UART từ các thiết bị bên ngoài, thường cần đánh thức chip khi dữ liệu đầu vào có sẵn. Bộ vi xử lý UART chứa một tính năng cho phép đánh thức chip từ chế độ ngủ nhẹ khi có một số lượng cạnh dương trên chân RX được nhìn thấy. Số lượng cạnh dương này có thể được thiết lập bằng cách sử dụng hàm uart_set_wakeup_threshold().
Lưu ý rằng ký tự kích hoạt đánh thức (và bất kỳ ký tự nào trước đó) sẽ không được nhận bởi UART sau khi đánh thức. Điều này có nghĩa là thiết bị bên ngoài thường cần gửi một ký tự bổ sung đến ESP32 để kích hoạt đánh thức trước khi gửi dữ liệu. Hàm esp_sleep_enable_uart_wakeup()có thể được sử dụng để kích hoạt nguồn đánh thức này.
