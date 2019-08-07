# Chirp for Arduino

*Version 3.3.0, August 2019*

## Overview

Chirp is a library enabling Arduino-based devices to send and receive data using sound. You'll need:

 * A compatible Arduino board
 * A digital I2S MEMS microphone
 * A digital I2S amplifier and compatible speaker

For sound input you will need a digital MEMS microphone such as the SPH0645 or ICS-43434. (Not necessary for the Nano 33 Sense as it comes with an on board microphone)
For sound output it is recommended to use a digital I2S output such as the UDA1334A or MAX98357A connected to a compatible speaker.

You can quickly test the sound input by playing random chirps from the [Developer Hub](https://developers.chirp.io).
The easiest way to test the sound output would be to use Chirp on the [command line](https://developers.chirp.io/docs/tutorials/command-line) to receive data from the Arduino.

## Supported hardware

Send and receive capabilities

 * Arduino Nano 33 Sense
 * Microsoft MXChip
 * ESP32

Send only

 * Arduino MKRZero
 * Arduino Vidor 4000
 * Genuino Zero
 * MKR Fox 1200
 * MKR1000 WiFi

## Installation

Chirp is written for the Arduino IDE versions 1.8.6 and above.

Install ChirpSDK as a library. For instructions, see

[http://arduino.cc/en/Guide/Libraries](http://arduino.cc/en/Guide/Libraries)

Once installed, you can access the example programs from the menu :

```File > Examples > ChirpSDK > Example ```

and you can include the headers to use Chirp in your own code by adding :

```#include "chirp_connect.h"```

## Usage

To set up the Chirp SDK, initialise and configure with your app key,
secret and config from the [Developer Hub](https://developers.chirp.io).

*Note* You must select the `arduino` protocol from the dropdown menu, when
selecting your chirp configuration.

    chirp = new_chirp_connect(APP_KEY, APP_SECRET);
    if (chirp == NULL) {
        Serial.println("Chirp initialisation failed.");
        return;
    }

Then set any required callbacks and start the SDK running.

    chirp_connect_error_code_t err = chirp_connect_set_config(chirp, APP_CONFIG);
    if (err != CHIRP_CONNECT_OK)
        return;

    chirp_connect_callback_set_t callbacks = {0};
    callbacks.on_received = onReceivedCallback;
    err = chirp_connect_set_callbacks(chirp, callbacks);
    if (err != CHIRP_CONNECT_OK)
        return;

    err = chirp_connect_set_callback_ptr(chirp, chirp);
    if (err != CHIRP_CONNECT_OK)
        return;

    err = chirp_connect_set_input_sample_rate(chirp, input_sample_rate);
    if (err != CHIRP_CONNECT_OK)
        return;
    err = chirp_connect_set_output_sample_rate(chirp, output_sample_rate);
    if (err != CHIRP_CONNECT_OK)
        return;

    err = chirp_connect_start(chirp);
    if (err != CHIRP_CONNECT_OK)
        return;

## Callbacks

The received data is passed back to the `onReceivedCallback` function. If the payload pointer is null then there has been an error decoding the data.

    void
    onReceivedCallback(void *chirp, uint8_t *payload, size_t length, uint8_t channel) {
        if (payload) {
            char *hexString = chirp_connect_as_string(chirp, payload, length);
            Serial.printf("Received data = %s\n", hexString);
            chirp_connect_free(hexString);
        } else {
            Serial.println("Decode failed.");
        }
    }

A complete list of callbacks is shown below.

    void onStateChangedCallback(void *ptr, chirp_connect_state_t old_state, chirp_connect_state_t new_state) {
        // Put here what you want to do when the SDK's state is changing.
    }

    void onSendingCallback(void *ptr, uint8_t *bytes, size_t length, uint8_t channel) {
        // Put here what you want to do when the SDK starts to send some data.
    }

    void onSentCallback(void *ptr, uint8_t *bytes, size_t length, uint8_t channel) {
        // Put here what you want to do when the SDK has sent some data.
    }

    void onReceivingCallback(void *ptr, uint8_t *bytes, size_t length, uint8_t channel) {
        // Put here what you want to do when the SDK starts receiving some data.
    }

    void onReceivedCallback(void *ptr, uint8_t *bytes, size_t length, uint8_t channel) {
        // Put here what you want to do when the SDK has received some data.
    }

    // If you don't set all the callbacks, make sure the unused callbacks are set to NULL.
    chirp_connect_callback_set_t callbacks_set = {
        .on_state_changed = on_state_changed_callback,
        .on_sending = on_sending_callback,
        .on_sent = on_sent_callback,
        .on_receiving = on_receiving_callback,
        .on_received = on_received_callback
    };
    err = chirp_connect_set_callbacks(chirp, callbacks_set);
    if (err != CHIRP_CONNECT_OK)
    {
        const char *error_string = chirp_connect_error_code_to_string(err);
        printf("%s\n", error_string);
    }


## Payloads

A Chirp payload is simply an array of bytes. You can send a random data payload to the speakers like so.

    size_t payload_length = chirp_connect_get_max_payload_length(chirp);
    uint8_t *payload = chirp_connect_random_payload(chirp, &payload_length);

    err = chirp_connect_send(chirp, payload, payload_length);
    if (err != CHIRP_CONNECT_OK) {
        const char *error_string = chirp_connect_error_code_to_string(err);
        printf("%s\n", error_string);
    }

Or you can easily send an ASCII string

    char *identifier = "hello";
    err = chirp_connect_send(chirp, (uint8_t *)identifier, strlen(identifier));
    if (err != CHIRP_CONNECT_OK) {
        const char *error_string = chirp_connect_error_code_to_string(err);
        printf("%s\n", error_string);
    }

## Processing

To process audio data from the microphone, and fill the output buffer with audio data, call the following functions with data periodically.

    err = chirp_connect_process_input(chirp, input_buffer, input_buffer_length);

    err = chirp_connect_process_output(chirp, output_buffer, output_buffer_length);

***

All content copyright &copy; Asio Ltd, 2013-2019. All rights reserved.
