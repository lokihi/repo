# repo
import RPi.GPIO as GPIO
import matplotlib.pyplot as plt
import time
#Инициализация
dac = [26, 19, 13, 6, 5, 11, 9, 10]
leds = [21, 20, 16, 12, 7, 8, 25, 24]
bits = len(dac)
levels = 2**bits
maxVoltage = 3.3
troykaModule = 17
comparator = 4
comparatorvalue = 1
list_of_values = []
value = 0
#Настройка
GPIO.setmode(GPIO.BCM)
GPIO.setup(dac, GPIO.OUT, initial = GPIO.LOW)
GPIO.setup(leds, GPIO.OUT, initial = GPIO.LOW)
GPIO.setup(troykaModule, GPIO.OUT, initial=GPIO.HIGH)
GPIO.setup(comparator, GPIO.IN)
#Объявление функций
def dec2bin(decimal):
    return [int(bit) for bit in bin(decimal)[2:].zfill(bits)]

def bin2dac(value):
    signal = dec2bin(value)
    GPIO.output(dac, signal)
    return signal

def bin2led(value):
    signal = dec2bin(value)
    GPIO.output(leds, signal)
    return signal

def adc():
    value = 2**(bits - 1)
    for i in range(1, 8):
        signal = bin2dac(value)
        time.sleep(0.001)
        comparatorvalue = GPIO.input(comparator)
        if comparatorvalue == 1:
            value += (2**(bits - i - 1))
        else:
            value -= (2**(bits - i - 1))
    value -= (value % 2)
    signal = bin2dac(value)
    return value
#Эксперимент
try:
    #Зарядка
    GPIO.output(troykaModule, 1)
    vremya = time.time()
    print("Идёт зарядка")
    while value <= 250:
        value = adc()
        list_of_values.append(value)
        bin2led(value)
    #Разрядка
    GPIO.output(troykaModule, 0)
    print("Идёт разрядка")
    while value >= 1:
        value = adc()
        list_of_values.append(value)
        bin2led(value)
    #Обработка данных
    print("Эксперимент завершён. Обработка данных.")
    vremya = time.time() - vremya
    freq = 1/(vremya/len(list_of_values))
    #Построение графика
    plt.plot(list_of_values)
    plt.show()
    #Запись результатов в файл
    list_of_values = [str(i) for i in list_of_values]
    with open("data.txt", "w") as file:
        file.write("\n".join(list_of_values))
    with open("settings.txt", "w") as file:
        file.write("Время экспенримента: {:.3f} с\n".format(vremya))
        file.write("Единица напряжения: {:.3f} В".format(maxVoltage/2**bits))

except KeyboardInterrupt:
    print("The program was stopped by the keyboard")
else:
    print("No exceptions")
finally:
    GPIO.output(dac, GPIO.LOW)
    GPIO.cleanup()
    print("GPIO cleanup completed, your majesty!")
