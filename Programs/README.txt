#import Header Files
import RPi.GPIO as GPIO
import time
import pygame
import qrcode
from gpiozero.pins.pigpio import PiGPIOFactory
from gpiozero import Servo
import random
from gpiozero import DistanceSensor

#GPIO Setmodes
GPIO.setmode(GPIO.BCM)
factory = PiGPIOFactory()
pygame.init()

#Pins Connected To GPIO Pins
Motor_1_Roller_IN1 = 17 #Connect 17 GPIO Pin 
Motor_1_Roller_IN2 = 27 #Connect 27 GPIO Pin
Motor_1_Roller_EN  = 22 #Connect 22 GPIO Pin 
Motor_2_Roller_IN1 = 2  #Connect 2 GPIO Pin 
Motor_2_Roller_IN2 = 3  #Connect 3 GPIO Pin 
Motor_2_Roller_EN  = 4  #Connect 4 GPIO Pin 
Motor_3_Roller_IN1 = 10 #Connect 10 GPIO Pin 
Motor_3_Roller_IN2 = 9  #Connect 9 GPIO Pin 
Motor_3_Roller_EN  = 11 #Connect 11 GPIO Pin 
Motor_4_Roller_IN1 = 23 #Connect 23 GPIO Pin 
Motor_4_Roller_IN2 = 24 #Connect 24 GPIO Pin 
Motor_4_Roller_EN  = 25 #Connect 25 GPIO Pin 
Motor_5_IN1 = 8 #Connect 8 GPIO Pin 
Motor_5_IN2 = 7 #Connect 7 GPIO Pin 
Motor_5_EN  = 1 #Connect 1 GPIO Pin 
Motor_6_IN1 = 16 #Connect 16 GPIO Pin 
Motor_6_IN2 = 20 #Connect 20 GPIO Pin 
Motor_6_EN  = 21 #Connect 21 GPIO Pin 
Pump_Motor_1 = 0 #Connect 0 GPIO Pin 
Pump_Motor_2 = 5 #Connect 5 GPIO Pin 
Servo_1_Pin = 12 #Connect 12 GPIO Pin 
Servo_2_Pin = 18 #Connect 18 GPIO Pin 
Ultrasonic_1_TRIG = 6  #Connect 6 GPIO Pin 
Ultrasonic_1_ECHO = 13 #Connect 13 GPIO Pin 
IR_Sensor_1 = 19 #Connect 19 GPIO Pin 
IR_Sensor_2 = 26 #Connect 26 GPIO Pin

#Colors For Display:
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
ORANGE = (255, 165, 0)

#Display Size and Caption:
WIDTH, HEIGHT = 500, 500
DISPLAY = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Fresh Juice Vending Machine")

#Display Font:
FONT = pygame.font.SysFont(None, 30)

#Input and Output Setting
GPIO.setup(Motor_1_Roller_IN1,GPIO.OUT) #Motor 1 IN1 As Ouput
GPIO.setup(Motor_1_Roller_IN2,GPIO.OUT) #Motor 1 IN2 As Ouput
GPIO.setup(Motor_1_Roller_EN,GPIO.OUT)  #Motor 1 Enable As Ouput
GPIO.setup(Motor_2_Roller_IN1,GPIO.OUT) #Motor 2 IN1 As Ouput
GPIO.setup(Motor_2_Roller_IN2,GPIO.OUT) #Motor 2 IN2 As Ouput
GPIO.setup(Motor_2_Roller_EN,GPIO.OUT)  #Motor 2 Enable As Ouput
GPIO.setup(Motor_3_Roller_IN1,GPIO.OUT) #Motor 3 IN1 As Ouput
GPIO.setup(Motor_3_Roller_IN2,GPIO.OUT) #Motor 3 IN2 As Ouput
GPIO.setup(Motor_3_Roller_EN,GPIO.OUT)  #Motor 3 Enable As Ouput
GPIO.setup(Motor_4_Roller_IN1,GPIO.OUT) #Motor 4 IN1 As Ouput
GPIO.setup(Motor_4_Roller_IN2,GPIO.OUT) #Motor 4 IN2 As Ouput
GPIO.setup(Motor_4_Roller_EN,GPIO.OUT)  #Motor 4 Enable As Ouput
GPIO.setup(Motor_5_IN1,GPIO.OUT) #Motor 5 IN1 As Ouput                            
GPIO.setup(Motor_5_IN2,GPIO.OUT) #Motor 5 IN2 As Ouput
GPIO.setup(Motor_5_EN,GPIO.OUT)  #Motor 5 Enable As Ouput
GPIO.setup(Motor_6_IN1,GPIO.OUT) #Motor 6 IN1 As Ouput                            
GPIO.setup(Motor_6_IN2,GPIO.OUT) #Motor 6 IN2 As Ouput
GPIO.setup(Motor_6_EN,GPIO.OUT)  #Motor 6 Enable As Ouput
GPIO.setup(Pump_Motor_1,GPIO.OUT) #Pump Motor 1 As Ouput
GPIO.setup(Pump_Motor_2,GPIO.OUT) #Pump Motor 2 As Ouput
Servo_1 = Servo(Servo_1_Pin,pin_factory=factory) #Servo 1 Pin 
Servo_2 = Servo(Servo_2_Pin,pin_factory=factory) #Servo 2 Pin
GPIO.setup(Ultrasonic_1_TRIG,GPIO.OUT) #Ultrasonic Sensor Trig as Output
GPIO.setup(Ultrasonic_1_ECHO,GPIO.IN) #Ultrasonic Sensor Echo as Input

#PWM setup for Enable Pins
PWM1=GPIO.PWM(Motor_1_Roller_EN,100) #PWM signal For Motor 1
PWM2=GPIO.PWM(Motor_2_Roller_EN,100) #PWM signal For Motor 2
PWM3=GPIO.PWM(Motor_3_Roller_EN,100) #PWM signal For Motor 3
PWM4=GPIO.PWM(Motor_4_Roller_EN,100) #PWM signal For Motor 4
PWM5=GPIO.PWM(Motor_5_EN,100) #PWM signal For Motor 5
PWM6=GPIO.PWM(Motor_6_EN,100) #PWM signal For Motor 6
PWM1.start(0) #Start from Zero For Motor 1
PWM2.start(0) #Start from Zero For Motor 2
PWM3.start(0) #Start from Zero For Motor 3
PWM4.start(0) #Start from Zero For Motor 4
PWM5.start(0) #Start from Zero For Motor 5
PWM6.start(0) #Start from Zero For Motor 6

#Ultrasonic Sensor Setup:
Max_Distance= 10 #Maximum Distance

#Ultrasonic sensor:
def measure_distance():
    GPIO.output(Ultrasonic_1_TRIG, GPIO.HIGH) #Set Trigger Pin as High
    time.sleep(0.00001) #Delay
    GPIO.output(Ultrasonic_1_TRIG, GPIO.LOW) #Set Trigger Pin as Low
    start_time = time.time() #Set Start Time
    while GPIO.input(Ultrasonic_1_ECHO) == 0: #While Loop
        start_time = time.time() #Start Time 
    while GPIO.input(Ultrasonic_1_ECHO) == 1: #While Loop
        end_time = time.time() #End Time
    pulse_duration = end_time - start_time #Set Pulse Duration
    distance = pulse_duration * 17150 #Set Distance using Pulse Duration
    distance = round(distance, 2) #Round The Distance In 2
    return distance #Return Distance

#Display and Payment Conditions:
def Step_One():
    DISPLAY.fill(WHITE) #Display White Background
    text = FONT.render("Fresh Juice Vending Machine", True, BLACK) #Show the Text in display
    DISPLAY.blit(text, (100, 100)) #Text Size and Position
    pygame.display.update() #Update The Display
    text = FONT.render("Enter the No Of Juices", True, BLACK) #Show The Text in Display
    DISPLAY.blit(text, (120, 120)) #Text Size and Position
    pygame.display.update() #update Display
    run_vending_machine() #Call Function
    
def select_oranges():
    price_per_orange_juice = 20 #Price Per Orange Juice
    number_of_orange_juices = int(input("Enter the number of oranges juice you want to buy: ('1,2,3,4,5')")).lower() #Select No of Juices are Buy
    Water=input("Select (Normal Water/Ice Water)").lower() #Select Water
    Sugar=input("Enter the Sugar (Full Sugar/Half Sugar/No Sugar)").lower() #Select Sugar Level
    total_cost = number_of_orange_juices * price_per_orange_juice #Total Cost
    return total_cost #Return to Total Cost

def generate_qr_code(amount):
    upi_id = "sanjaybalaji2006@oksbi" #Your UPI I'd
    qr_data = f"upi://pay?pa={upi_id}&pn=FreshJuiceVendingMachine&am={amount}&cu=INR" #QRCode Data
    qr_img = qrcode.make(qr_data) #Make QRCode
    qr_img.save("qr_code.png") #Save QRCode
    print("Please scan the QR code to pay.") #Display Scan QRCode
    return qr_data  # Return QR code data for verification

def check_payment(qr_data_scanned):
    payment_verified = check_payment(qr_data_scanned) #Fill Functions In Variable
    if qr_data_scanned and payment_verified: #Condition True
        print("Payment successful. Machine is starting to make juice.") #Display Payment Successful
    else: #Condtion Fail
        print("Payment verification failed. Please try again.") #Display payment Fail
        time.sleep(3) #Wait 3 Seconds
        return draw_frame1() #call Function

def run_vending_machine():
    print("Welcome to the fresh juice Vending Machine!") #Display Welcome
    total_cost = select_oranges() #Set Function in Variable
    
    print(f"Total cost is: INR {total_cost}") #Display The Total Cost
    DISPLAY.fill(WHITE) #Displa White Backgroud
    text = FONT.render("scan the QRCode and pay", True, BLACK) #Show The Text In Display
    DISPLAY.blit(text, (100, 480)) #Text Size and Position
    qr_img = pygame.image.load("qr_code.png") #Load The QRCode in Display
    DISPLAY.blit(qr_img, (0, 0)) #Image Size and Position
    pygame.display.update() #Update Display
    qr_data = generate_qr_code(total_cost) #Set Function in Variable
    qr_data_scanned = False  # Flag to track if QR code has been scanned
    while not qr_data_scanned: #While Loop
        for event in pygame.event.get(): #For Loop
            if event.type == pygame.MOUSEBUTTONDOWN: #Condition True
                mouse_pos = pygame.mouse.get_pos() #Set Variable
                if 0 <= mouse_pos[0] <= 500 and 0 <= mouse_pos[1] <= 500: #Condition True
                    qr_data_scanned = True #set Variable
    DISPLAY.fill(WHITE) #White Background
    text = FONT.render("Scan the QR Code and pay", True, BLACK) #Show Text in Display
    DISPLAY.blit(text, (100, 480)) #Text Size and Position
    qr_img = pygame.image.load("qr_code.png") # Load the Image
    DISPLAY.blit(qr_img, (0, 0)) #Image Size and Position
    pygame.display.update() #Update Display
    payment_verified = check_payment(qr_data_scanned) #Set Function in Variable
    if payment_verified: #Condition True
        print("Payment successful. Machine is starting to make juice.") #Display Payment Successful
    else: #Condition Fail
        print("Payment verification failed. Please try again.") #Display Payment Fail
        time.sleep(3) #Wait 3 Seconds
        return run_vending_machine() #Return to Function

def Payment_main():
    running = True #Set Variable True
    while running: #While Loop
        for event in pygame.event.get(): #For Loop
            if event.type == pygame.QUIT: #Condition True
                running = False #Set Variable False
        Step_One() #Call Function
        time.sleep(1) #Wait 1 Seconds
    pygame.quit() #pygame Exit

#Glass Dispenser Element:
def Glass_Dispenser_Element():
    GPIO.output(Motor_5_IN1,GPIO.HIGH) #Motor 5 IN1 High
    GPIO.output(Motor_5_IN2,GPIO.LOW)  #Motor 5 IN2 Low
    PWM5.ChangeDutyCycle(100) #Change Duty Cycle to 100 For Full Speed
    print("Running in Clockwise") #Display Motor Running
    time.sleep(6) #Wait For 6 Seconds
    GPIO.output(Motor_5_IN1,GPIO.LOW)  #Motor 5 IN1 Low
    GPIO.output(Motor_5_IN2,GPIO.LOW)  #Motor 5 IN2 Low
    PWM5.ChangeDutyCycle(0) #Change Duty Cycle to 0 For Stop
    print("stop") #Display Motor As Stop
    time.sleep(2) #Wait For 2 Seconds
    GPIO.output(Motor_5_IN1,GPIO.LOW)  #Motor 5 IN1 Low
    GPIO.output(Motor_5_IN2,GPIO.HIGH) #Motor 5 IN2 High
    PWM5.ChangeDutyCycle(100) #Change Duty Cycle to 100 For Full Speed
    print("Running in AntiClockwise") #Display Motor Running
    time.sleep(6) #Wait For 6 Seconds
    GPIO.output(Motor_5_IN1,GPIO.LOW)  #Motor 5 IN1 Low
    GPIO.output(Motor_5_IN2,GPIO.LOW)  #Motor 5 IN2 Low
    PWM5.ChangeDutyCycle(0) #Change Duty Cycle to 0 For Stop
    print("stop") #Display Motor As Stop
    time.sleep(2) #Wait For 2 Seconds

#Water Dispenser Element:
def Water_Dispenser_Element():
    if Water==1: #check 1 is Selected
        print("Selected Normal Water") #Display Normal Water is Selected
        time.sleep(1) #Wait 1 Second
        GPIO.output(Pump_Motor_1,GPIO.HIGH) #Pump Motor 1 as High
        print("Pump Motor 1 ON") #Display Pump Motor 1 is ON
        time.sleep(4) #Wait 4 Seconds
        GPIO.output(Pump_Motor_1,GPIO.LOW) #Pump Motor 1 as Low
        print("Pump Motor 1 OFF") #Display Pump Motor 1 is OFF
        time.sleep(2) #wait 2 Seconds
    elif Water_Condition==2: #check 2 is Selected
        print("Selected Ice Water") #Display Ice Water is Selected
        GPIO.output(Pump_Motor_2,GPIO.HIGH) #Pump Motor 2 as High
        print("Pump Motor 2 ON") #Disply Pump Motor 2 is ON
        time.sleep(4) #Wait 4 Seconds
        GPIO.output(Pump_Motor_2,GPIO.LOW) #Pump Motor 2 as Low
        print("Pump Motor 2 OFF") #Display Pump Motor 2 is OFF
        time.sleep(2) #Wait 2 Seconds
    
#Orange Storage Tank Element:
def Orange_Storage_Tank_Element():
    print("Motor Rotation for 30 Degree") #Display Motor Rotated 30 Degree
    Servo_1.value = 0.279 #Setting Value For Servo
    time.sleep(0.549) #Wait 0.549 Seconds
    
#Roller Element:
def Roller_Element():
    print("Motor Started to Rotate") #Display Motors Are Started
    GPIO.output(Motor_1_Roller_IN1,GPIO.HIGH) #Motor 1 IN1 High 
    GPIO.output(Motor_1_Roller_IN2,GPIO.LOW)  #Motor 1 IN2 Low
    PWM1.ChangeDutyCycle(100) #Change Duty Cycle to 100 For Full Speed
    GPIO.output(Motor_2_Roller_IN1,GPIO.HIGH) #Motor 2 IN1 High 
    GPIO.output(Motor_2_Roller_IN2,GPIO.LOW)  #Motor 2 IN2 Low
    PWM2.ChangeDutyCycle(100) #Change Duty Cycle to 100 For Full Speed
    GPIO.output(Motor_3_Roller_IN1,GPIO.HIGH) #Motor 3 IN1 High 
    GPIO.output(Motor_3_Roller_IN2,GPIO.LOW)  #Motor 3 IN2 Low
    PWM3.ChangeDutyCycle(100) #Change Duty Cycle to 100 For Full Speed
    GPIO.output(Motor_4_Roller_IN1,GPIO.HIGH) #Motor 4 IN1 High 
    GPIO.output(Motor_4_Roller_IN2,GPIO.LOW)  #Motor 4 IN2 Low
    PWM4.ChangeDutyCycle(100) #Change Duty Cycle to 100 For Full Speed
    time.sleep(10) #Wait For 10 Seconds
    print("Motor Stop the Rotate") #Display Motor Are Stoped
    GPIO.output(Motor_1_Roller_IN1,GPIO.LOW) #Motor 1 IN1 Low
    GPIO.output(Motor_1_Roller_IN2,GPIO.LOW) #Motor 1 IN2 Low
    PWM1.ChangeDutyCycle(0) #Change Duty Cycle to 0 For Stop
    GPIO.output(Motor_2_Roller_IN1,GPIO.LOW) #Motor 2 IN1 Low
    GPIO.output(Motor_2_Roller_IN2,GPIO.LOW) #Motor 2 IN2 Low
    PWM2.ChangeDutyCycle(0) #Change Duty Cycle to 0 For Stop
    GPIO.output(Motor_3_Roller_IN1,GPIO.LOW) #Motor 3 IN1 Low
    GPIO.output(Motor_3_Roller_IN2,GPIO.LOW) #Motor 3 IN2 Low
    PWM3.ChangeDutyCycle(0) #Change Duty Cycle to 0 For Stop
    GPIO.output(Motor_4_Roller_IN1,GPIO.LOW) #Motor 4 IN1 Low
    GPIO.output(Motor_4_Roller_IN2,GPIO.LOW) #Motor 4 IN2 Low
    PWM4.ChangeDutyCycle(0) #Change Duty Cycle to 0 For Stop

#Rotory Conveyor Rotatiion 1
def Rotory_Conveyor_1():

#Rotory Conveyor Rotatiion 2
def Rotory_Conveyor_2():
    
#Open And Closing Door:
def Rack_And_Pinion_Door():
    
#Main Program:
def Main_Program():
    while True: #While Loop
        Payment_main() #Call Function
        Glass_Dispenser_Element() #Call Function
        time.sleep(2) #Delay
        while True: #While Loop
            if IR_Sensor_1 == HIGH: #Condition True
                break #Exit Loop
            else: #Condtion False
                Glass_Dispenser_Element() #Return To Function
        time.sleep(2) #Delay
        Rotory_Conveyor_1() #Call Function
        while True: #Whilr Loop
            if IR_Sensor_2 == HIGH: #Condition True
                break #Exit Loop
            else: #Condition False
                return Rotory_Conveyor_1()
        time.sleep(2)
        Orange_Storage_Tank_Element()
        Roller_Element()
        Water_Dispenser_Element()
        Dist=measure_distance()
        while True:
            if  Dist <= Max_Distance:
                break 
            else:
                Water_Dispenser_Element()
        time.sleep(2)
        Rotory_Conveyor_2()
        
Main_Program()
