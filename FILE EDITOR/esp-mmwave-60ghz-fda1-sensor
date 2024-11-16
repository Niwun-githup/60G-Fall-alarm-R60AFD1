#include "esphome.h"

using namespace esphome;

class UARTSensor : public Component, public UARTDevice {
  public:
    UARTSensor(UARTComponent *parent) : UARTDevice(parent) {}

    // เซ็นเซอร์ที่ใช้สำหรับข้อมูลต่าง ๆ
    Sensor* presence_sensor = new Sensor();
    Sensor* motion_sensor = new Sensor();
    Sensor* activity_sensor = new Sensor();
    Sensor* fall_sensor = new Sensor();
    Sensor* fall_det_sw = new Sensor();
    Sensor* fall_det_sens = new Sensor();

    // เพิ่มเซ็นเซอร์สำหรับสถานะการอยู่อาศัยที่ไม่เคลื่อนไหว
    Sensor* stationary_residence_sensor = new Sensor();

    void setup() {
        // การตั้งค่าเริ่มต้น หากมี
    }

    std::vector<int> bytes;

    void loop() override {
        while (available()) {
            bytes.push_back(read());

            // End of Frame is 0x54 0x43
            if (bytes[bytes.size() - 2] == 0x54 && bytes[bytes.size() - 1] == 0x43) {
                processPacket();
                bytes.clear();
            }
        }
    }

    void processPacket() {
        std::string str = "";

        // ตัวอย่างการตรวจสอบสถานะการอยู่อาศัยที่ไม่เคลื่อนไหว
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x83) && (bytes[3] == 0x05) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            int stationary_residence_value = int(bytes[6]); // 0x00: ไม่มีสถานะการอยู่อาศัยที่ไม่เคลื่อนไหว, 0x01: มีสถานะการอยู่อาศัยที่ไม่เคลื่อนไหว

            // ส่งค่าไปยังเซ็นเซอร์
            stationary_residence_sensor->publish_state(stationary_residence_value);

            // Log ข้อมูล
            // str = "Stationary Residence State: " + hexStr(stationary_residence_value);
            // ESP_LOGI("Stationary Residence State Frame", str.c_str());

            return;
        }

        // ตรวจสอบสถานะการอยู่อาศัยที่ไม่เคลื่อนไหว (Presence)
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x80) && (bytes[3] == 0x01) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            int presence_sensor_value = int(bytes[6]);
            presence_sensor->publish_state(presence_sensor_value);
            return;
        }

        // ตรวจสอบสถานะการเคลื่อนไหว (Motion)
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x80) && (bytes[3] == 0x02) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            if (bytes[6] == 0x02) {
                motion_sensor->publish_state(1); // มีการเคลื่อนไหว
            } else if (bytes[6] == 0x01) {
                motion_sensor->publish_state(0); // ไม่มีการเคลื่อนไหว
            }
            return;
        }

        // ตรวจสอบระดับกิจกรรม (Activity)
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x80) && (bytes[3] == 0x03) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            int activity_sensor_value = int(bytes[6]);
            activity_sensor->publish_state(activity_sensor_value);
            return;
        }

        // ตรวจสอบการตรวจจับการล้ม (Fall Detection)
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x83) && (bytes[3] == 0x01) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            int fall_sensor_value = int(bytes[6]);
            fall_sensor->publish_state(fall_sensor_value);
            return;
        }

        // ตรวจสอบการตั้งค่าความไวในการตรวจจับการล้ม (Fall Sensitivity)
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x83) && (bytes[3] == 0x0D) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            int fall_det_sens_state = int(bytes[6]);
            fall_det_sens->publish_state(fall_det_sens_state);
            return;
        }

        // ตรวจสอบการเปิด/ปิดฟังก์ชันการตรวจจับการล้ม (Fall Detection Switch)
        if ((bytes[0] == 0x53) && (bytes[1] == 0x59) && (bytes[2] == 0x83) && (bytes[3] == 0x00) && (bytes[4] == 0x00) && (bytes[5] == 0x01)) {
            int fall_det_sw_state = int(bytes[6]);
            fall_det_sw->publish_state(fall_det_sw_state);
            return;
        }
    }

    char hexmap[16] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

    std::string hexStr(int data) {
        std::string s = "    ";
        s[0] = '0';
        s[1] = 'x';
        s[2] = hexmap[(data & 0xF0) >> 4];
        s[3] = hexmap[data & 0x0F];
        return s;
    }
};
