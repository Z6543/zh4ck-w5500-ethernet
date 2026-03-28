# zh4ck-w5500-ethernet

ESP-IDF component for W5500 Ethernet over SPI on the [Tanmatsu](https://github.com/Nicolai-Electronics/tanmatsu-launcher)  badge's J4 PMOD connector.

## Usage

Drop this repository into your project's `components/` directory, then call from your app:

```c
#include "zh4ck_w5500_ethernet.h"

// esp_netif_init() and esp_event_loop_create_default() must be called first
esp_err_t ret = ethernet_init();
if (ret == ESP_ERR_NOT_FOUND) {
    // W5500 not connected — non-fatal, continue without Ethernet
} else if (ret != ESP_OK) {
    ESP_LOGE(TAG, "Ethernet init failed: %s", esp_err_to_name(ret));
}

// Later, check link + IP status
if (ethernet_connected()) {
    // ready to use
}
```

## API

```c
// Initialize W5500 Ethernet over SPI on the J4 PMOD connector.
// Requires esp_netif and default event loop to be initialized first.
// Returns ESP_OK on success, ESP_ERR_NOT_FOUND if no W5500 detected.
esp_err_t ethernet_init(void);

// Returns true if Ethernet link is up and an IP address has been acquired.
bool ethernet_connected(void);
```

## Hardware

Targets the J4 PMOD connector on Tanmatsu hardware. Pins are fixed:

| Signal | GPIO | Pad   |
|--------|------|-------|
| SCK    | 2    | MTCK  |
| MOSI   | 3    | MTDI  |
| MISO   | 5    | MTDO  |
| CS     | 4    | MTMS  |
| INT    | 15   | SAO_IO1 |
| RST    | 34   | SAO_IO2 |

SPI host: `SPI2_HOST` at 12 MHz.

## Dependencies

Declared in `CMakeLists.txt`: `driver`, `esp_eth`, `esp_event`, `esp_netif`.
