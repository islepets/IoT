# Интернет вещей. Лабораторная №2

## Сканер I2C

I²C - протокол для связи между микросхемами внутри одного устройства. Он использует всего две линии связи: 
SDA(данные) и SDL(тактовый сигнал). Устройства к этим линиям подключаются параллельно. Каждое устройство 
имеет 7-ми битный адрес.

• Для выполнения работы нужно подключить ADXL345 и BME280 и спомощью выводимой таблицы определить 
адреса этих устройств.


## Акселерометр

Акселерометр - измеряет ускорение по трем осям: X, Y и Z. Внутри чипа находятся микроскопические емкостные сенсоры,
которые при ускорении смещаются, изменяя емкость. Электроника преобразует эти изменения в цифровые значения.

```
esp_err_t adxl345_write_reg(uint8_t reg_addr, uint8_t data)
{
    i2c_cmd_handle_t cmd = i2c_cmd_link_create();
    
    i2c_master_start(cmd);  // START условие
    i2c_master_write_byte(cmd, (ADXL345_ADDR << 1) | I2C_MASTER_WRITE, true); // Адрес + запись
    i2c_master_write_byte(cmd, reg_addr, true);  // Адрес регистра
    i2c_master_write_byte(cmd, data, true);      // Данные для записи
    i2c_master_stop(cmd);   // STOP условие
    
    err = i2c_master_cmd_begin(I2C_MASTER_NUM, cmd, pdMS_TO_TICKS(1000));
    i2c_cmd_link_delete(cmd);
    return err;
}
```
• Процесс записи: START → Адрес+W → Регистр → Данные → STOP

```
esp_err_t adxl345_read_regs(uint8_t reg_addr, uint8_t *data, size_t len)
{
    // Сначала записываем адрес регистра для чтения
    i2c_master_start(cmd);
    i2c_master_write_byte(cmd, (ADXL345_ADDR << 1) | I2C_MASTER_WRITE, true);
    i2c_master_write_byte(cmd, reg_addr, true);

    // Затем читаем данные
    i2c_master_start(cmd);  // Повторный START
    i2c_master_write_byte(cmd, (ADXL345_ADDR << 1) | I2C_MASTER_READ, true);
    if (len > 1) {
        i2c_master_read(cmd, data, len - 1, I2C_MASTER_ACK);  // ACK для всех кроме последнего
    }
    i2c_master_read_byte(cmd, data + len - 1, I2C_MASTER_NACK); // NACK для последнего
    i2c_master_stop(cmd);
}
```
• Процесс чтения: START → Адрес+W → Регистр → Повторный START → Адрес+R → Чтение данных → STOP