#!/usr/bin/env python
# coding: utf-8

# In[1]:


import cv2
import mediapipe as mp

mp_hands = mp.solutions.hands
mp_drawing = mp.solutions.drawing_utils

hands = mp_hands.Hands(
    static_image_mode=False,
    max_num_hands=2,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
)

cap = cv2.VideoCapture(0)

while cap.isOpened():
    success, frame = cap.read()
    if not success:
        print("Gagal membaca dari kamera.")
        break

    image = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    image.flags.writeable = False

    results = hands.process(image)

    image.flags.writeable = True
    image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mp_drawing.draw_landmarks(
                image, hand_landmarks, mp_hands.HAND_CONNECTIONS
            )

    cv2.imshow('Kerangka Tangan - Tekan ESC untuk keluar', image)

    if cv2.waitKey(5) & 0xFF == 27:
        break

cap.release()
cv2.destroyAllWindows()
hands.close()


# In[ ]:
import os
os.environ['PYGAME_HIDE_SUPPORT_PROMPT'] = "hide"

import pygame
from pygame.locals import *
import time
import random
import mediapipe as mp
import pyautogui
import cv2



# Class

class Basket(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
    def draw(self, window):
        window.blit(basket_img, (self.x, self.y))
        

def text_objects(text, font):
    textSurface = font.render(text, True, black)
    return textSurface, textSurface.get_rect()

def message_to_screen(msg, x, y, size):
    regText = pygame.font.Font("freesansbold.ttf", size)
    textSurf, textRect = text_objects(msg, regText)
    textRect.center = (x, y)
    screen.blit(textSurf, textRect)

# Inisialisasi MediaPipe Hands
mp_hands = mp.solutions.hands
hands = mp_hands.Hands()
mp_draw = mp.solutions.drawing_utils

# Inisialisasi Pygame
pygame.init()
black = (0, 0, 0)

bg = pygame.image.load('background_2.jpg')
bg = pygame.transform.scale(bg, (800, 500))
basket_img = pygame.image.load('basket.png')
basket_img = pygame.transform.scale(basket_img, (150, 100))

apple_count = 0

SCREEN_WIDTH = 800
SCREEN_HEIGHT = 500
screen = pygame.display.set_mode((SCREEN_WIDTH,SCREEN_HEIGHT))

selected_apple = None


clock = pygame.time.Clock()

title_image = pygame.image.load('title_menu.png')
title_image = pygame.transform.scale(title_image, (800, 500)).convert()

bg_image = pygame.image.load('background_2.jpg')
bg_image = pygame.transform.scale(bg_image, (800, 500)).convert()

font = pygame.font.Font(None, 30)
med_font = pygame.font.Font(None, 50)
large_font = pygame.font.Font(None, 70)

loss_message = font.render('Anda Selesai Level Ini! silahkan tekan enter untuk main lagi tekan enter', True, (255, 0, 0))
loss_rect = loss_message.get_rect(center=(400, 250))

easy_message = med_font.render('LEVEL 1', True, (255, 255, 255)).convert_alpha()
easy_rect = easy_message.get_rect(center=(400, 400))
med_message = med_font.render('LEVEL 2', True, (255, 255, 255)).convert_alpha()
med_rect = med_message.get_rect(center=(400, 400))
hard_message = med_font.render('LEVEL 3', True, (255, 255, 255)).convert_alpha()
hard_rect = hard_message.get_rect(center=(400, 400))

con_message = font.render('Tekan Enter Untuk Bermain', True, (255, 0, 0)).convert_alpha()
con_rect = hard_message.get_rect(center=(320, 360))


overlay_surf = pygame.Surface((SCREEN_WIDTH, SCREEN_HEIGHT))
overlay_surf.fill((0, 0, 0))
overlay_surf.set_alpha(70)
overlay_surf = overlay_surf.convert_alpha()
overlay_rect = overlay_surf.get_rect()

#Halaman Utama Games
show_title_screen = True
fade_title_screen = False
difficulty = [['level_1', 1, 10], ['level_2', 3, 12], ['level_3', 5, 14]]
max_lives = 5                           
apples_reset = 12
current_difficulty = 0
title_fade_index = 255



# set title 
pygame.display.set_caption('EPICS- supported by IEEE') 


# Muat gambar tangan
hand_image = pygame.image.load('tangan.png')
hand_image = pygame.transform.scale(hand_image, (70, 70))
hand_rect = hand_image.get_rect()

# Warna dan Objek
white = (255, 255, 255)
obj_color = (0, 128, 255)
obj_pos = [SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2]
obj_radius = 20
is_dragging = False
drag_offset_x, drag_offset_y = 0, 0



# Mengatur kamera
cap = cv2.VideoCapture(0)

# Game loop
score = 0
basket = Basket(500 * 0.35, 800 - 160)
basket.x = 300
basket.y = 400
running = True
#last_x_index = 0
x_index =0


while show_title_screen or fade_title_screen:
        
        # Event Handling
        for event in pygame.event.get():
            # Player Closed Pygame
            if event.type == pygame.QUIT:
                running = False
                show_title_screen = False
            elif event.type == KEYDOWN:
                # Player Hit ESC to Quit
                if event.key == K_ESCAPE:
                    running = False
                    show_title_screen = False
                # Player Hit ENTER to Continue / Restart
                if event.key == K_RETURN:
                    show_title_screen = False
                    fade_title_screen = True
                    
                # Difficulty
                if event.key == K_UP:
                    if current_difficulty < len(difficulty) - 1:
                        current_difficulty += 1
                    else:
                        current_difficulty = 0
                if event.key == K_DOWN:
                    if current_difficulty == 0:
                        current_difficulty = len(difficulty) - 1
                    else:
                        current_difficulty -= 1

        # Fades Out The Title Screen
        if fade_title_screen:
            title_fade_index -= 10
            title_fade_index = max(title_fade_index, 0)
            title_image.set_alpha(title_fade_index)
            easy_message.set_alpha(title_fade_index)
            med_message.set_alpha(title_fade_index)
            hard_message.set_alpha(title_fade_index)
            if title_fade_index == 0:
                fade_title_screen = False
        
        # Displays the underlying background image and the title overay
        screen.blit(bg_image, (0, 0))
        screen.blit(title_image, (0, 0))

        # Displays the Difficulty Text
        if not fade_title_screen and show_title_screen:
            if difficulty[current_difficulty][0] == 'level_1':
                screen.blit(easy_message, easy_rect)
                screen.blit(con_message, con_rect)
                apple_count = difficulty [current_difficulty][1]
            elif difficulty[current_difficulty][0] == 'level_2':
                screen.blit(med_message, med_rect)
                screen.blit(con_message, con_rect)
                apple_count = difficulty [current_difficulty][1]
            elif difficulty[current_difficulty][0] == 'level_3':
                screen.blit(hard_message, hard_rect)
                screen.blit(con_message, con_rect)
                apple_count = difficulty [current_difficulty][1]

        pygame.display.update()
        clock.tick(60)
        
if difficulty[current_difficulty][0] == 'level_1':
    apple_img = pygame.image.load('strawberry.png')
    apple_img = pygame.transform.scale(apple_img, (60, 60))  
elif difficulty[current_difficulty][0] == 'level_2':
    apple_img = pygame.image.load('bomb.png')
    apple_img = pygame.transform.scale(apple_img, (60, 60))  
else :
    apple_img = pygame.image.load('banana.png')
    apple_img = pygame.transform.scale(apple_img, (60, 60))

#token = get_user_input()



while running:
    apples = [{'rect': apple_img.get_rect(center=(random.randint(50, SCREEN_WIDTH-50)
                                                , random.randint(50, SCREEN_HEIGHT-50)))
                                                , 'dragging': False, 'offset_x': 0
                                                , 'offset_y': 0
                                                , 'ID': i } for i in range(apple_count)]
    stop_game = False
    while not stop_game:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
                stop_game = True


            elif event.type == KEYDOWN:
                if event.key == K_RETURN:
                    stop_game = True
                    score = 0
                    # # Specify the URL you want to open
                    # clean_token = token.strip() 
                    # url = "https://epics-smile.id/dataprogress/"+clean_token+"/success"

                    # # Open the URL in the default web browser
                    # webbrowser.open(url)

        success, img = cap.read()
        if not success:
            continue

        img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        results = hands.process(img_rgb)
        
        if results.multi_hand_landmarks:
            for hand_landmarks in results.multi_hand_landmarks:
                index_tip = hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP]
                thumb_tip = hand_landmarks.landmark[mp_hands.HandLandmark.THUMB_TIP]
                x_index, y_index = int(index_tip.x * SCREEN_WIDTH), int(index_tip.y * SCREEN_HEIGHT)
                x_thumb, y_thumb = int(thumb_tip.x * SCREEN_WIDTH), int(thumb_tip.y * SCREEN_HEIGHT)

                # Update posisi gambar tangan
                # Membalik arah dari gerakan tangan
                if x_index >= (SCREEN_WIDTH/2):
                    x_index = (SCREEN_WIDTH/2)-(x_index - (SCREEN_WIDTH/2))
                else:
                    x_index = (SCREEN_WIDTH/2)+((SCREEN_WIDTH/2)-x_index)

                hand_rect.center = (x_index, y_index)
                

                if not is_dragging:
                    #print(1)
                    for apple in apples:
                        if apple['rect'].collidepoint(hand_rect.center):
                            apple['dragging'] = True
                            apple['offset_x'] = apple['rect'].x - x_index
                            apple['offset_y'] = apple['rect'].y - y_index
                            selected_apple = apple
                            print(selected_apple)
                            break
                    is_dragging =True
                else :
                    #print(2)
                    is_dragging = False
                
                if is_dragging:
                    #print(0)
                    if selected_apple and selected_apple['dragging']:
                        selected_apple['rect'].x = x_index + selected_apple['offset_x']
                        selected_apple['rect'].y = y_index + selected_apple['offset_y']
                                
                        if (selected_apple['rect'].x >=basket.x ) and (selected_apple['rect'].x <= basket.x+50 ):
                            if (selected_apple['rect'].y >=basket.y) and (selected_apple['rect'].y <=basket.y+50):
                                for i in apples :
                                    if i ["ID"]== selected_apple['ID']:
                                        apples.remove(i)
                                        score +=1
                                        is_dragging=False
                                        print(score)
                                        print(apple_count)
                                      

                mp_draw.draw_landmarks(img, hand_landmarks, mp_hands.HAND_CONNECTIONS)
        
        

        screen.blit(bg, (0,0))
        message_to_screen("Score: "+str(score), 50, 30, 20)
        basket.draw(screen)
        

        for apple in apples:
            screen.blit(apple_img, apple['rect'])

        if score ==  apple_count:
            screen.blit(loss_message, loss_rect)
            screen.blit(overlay_surf, overlay_rect)
        else :                          
            screen.blit(hand_image, hand_rect)

        pygame.display.update()
        cv2.imshow("Image", img)
        cv2.waitKey(1)
    
cap.release()
pygame.quit()
