# lerning
from re import M
import pygame
import pygame_gui   #pip install pygame_gui
import sys
import random

#pygame_gui에 관한 코드 자세한 설명은 https://pygame-gui.readthedocs.io/en/latest/index.html 이 쪽을 보면 더 좋을 듯
pygame.init()

WIDTH, HEIGHT = 1600, 900       #WIDTH와 HEIGHT 상수 설정
SCREEN = pygame.display.set_mode((WIDTH, HEIGHT))   #파이게임 화면 사이즈 설정

manager = pygame_gui.UIManager((1600, 900))
manager1 = pygame_gui.UIManager((1600, 900))
#pygame gui의 매니저 변수 설정 각각 text_input과 text_log manager

#img변수 설정
img = pygame.image.load("C:/Users/tlaeh/Downloads/111.jpg")     #파일 저장된 위치를 입력하기, 본인 컴퓨터에 맞춰서 설정 + \은 /으로 바꿔서 넣기
img = pygame.transform.scale(img, (1600,900))   #이미지 불러오기, img파일, 1600,900 사이즈로

text_input = pygame_gui.elements.UITextEntryLine(relative_rect=pygame.Rect((650, 800), (300, 50)), manager=manager,
                                               object_id='#main_text_line')
#숫자를 입력 받게 해주는 텍스트 박스 생성 코드, (650,800)는 위치/ (300,50)는 박스 크기
text_input.set_text_length_limit(4)     #text_input이 입력 받을 수 있는 길이 제한, 4글자
text_input.set_allowed_characters('numbers')    #받을 수 있는 문자의 종류 제한, 숫자

text_log = pygame_gui.elements.UITextBox(html_text="==========Log==========<body bgcolor='#DCDCDC'><font face='bahnschrift' color='#000000'><b>"
, relative_rect=pygame.Rect((70,400),(200,100)), wrap_to_height=True, manager=manager1, object_id='#log_text_box')
#로그를 보여주는 텍스트 박스 생성, 텍스트 박스 안의 텍스트 추가(html_text), 박스의 색상(<body bgcolor>), 폰트 종류(<font face>)&색상(<color>), 폰트 굵게(<b>)
#(70,400)은 위치, (200,100)박스 사이즈, 문자 추가 시 길이 늘어남(wrap_to_height=True)
text_log.disable()  #텍스트 박스 비활성화(= 글자 추가입력 불가능)
text_log.append_html_text("<br>   Num | S | B | Out   </b>")     #html_text추가                                    

clock = pygame.time.Clock()
#프레임 속도를 지정하는 clock객체 생성

conut = 0       #목숨
s = b = Out = 0 #볼카운트

alist = []      #컴퓨터가 설정한 임의 숫자 4개
for i in range(4):
    a = random.randint(0,9) #0~9까지 중 숫자 임의 추첨
    while a in alist :  #만약 중복된 숫자가 있을 시, 다시 추첨하기
        a = random.randint(0,9)
    alist.append(a)

#아래 47, 48은 정답 알려주는 코드, 지워야됨.
op = pygame.font.SysFont("bahnschrift", 10).render(f"{alist}", True, "black")
op_rect = op.get_rect(center=(650,70))

slist = []
event_1 = 0

def show_num(num):  #화면 출력 함수
    global conut       #전역변수 conut 불러오기
    conut += 1

    if conut == 11:     #숫자르 10번 입력했을 때
        end_game_lose() #end_game_lose()함수 불러오기

    while True:
        UI_REFRESH_RATE = clock.tick(60)/1000 #프로그램의 fps를 초당 최대 60로 설정
        for event in pygame.event.get():    #게임 실행하는 동안
            if event.type == pygame.QUIT:   #프로그램을 종료하는 이벤트 발생 시
                pygame.quit()
                sys.exit()
            if (event.type == pygame_gui.UI_TEXT_ENTRY_FINISHED and
                event.ui_object_id == '#main_text_line'):   
                #'#main_text_line'라는 오브젝트 이름을 가진 텍스트 박스(text_input)에 entry입력이 확인된다면
                game_num(event.text)    #입력된 숫자를 갖고 game_num()함수로 이동

            manager.process_events(event)   #text_input에 기록된 숫자(이벤트)를 이벤트 변수라고 저장

        SCREEN.blit(img,(0,0))  #img변수를 (0, 0)좌표 화면에 출력

        SCREEN.blit(op,op_rect) #op,op_rcet변수를 화면에 출력| 근데 이게 정답알려주는 코드, 나중에 지우기
        
        #아래는 화면에 출력될 텍스트 변수들 설정, 반복적인 내용임
        new_text = pygame.font.SysFont("bahnschrift", 100).render(f"{num}", True, "black")
        #폰트가 "bahnschrift", 크기가 100, 문구는 f"..."을 검은색으로 
        #변수는 {}안에 변수명을 쳐줘야됨
        new_text_rect = new_text.get_rect(center=(WIDTH/2, HEIGHT/2))
        #new_text가 출력될 위치 지정
        SCREEN.blit(new_text, new_text_rect)

        new_conut = pygame.font.SysFont("malgungothic", 25).render(f"남은 횟수 : {11-conut}", True, "black")
        #한글을 출력하는 경우 "malgungothic"폰트를 사용해야함. 안그럼 글씨가 깨지는 현상 발생
        new_conut_rect = new_conut.get_rect(center=(150,70))
        SCREEN.blit(new_conut,new_conut_rect)

        new_point = pygame.font.SysFont("bahnschrift", 40).render(f"S : {s} | B : {b} | Out : {Out}", True, "black")
        new_point_rect = new_point.get_rect(center=(1300,70))
        SCREEN.blit(new_point,new_point_rect)

        clock.tick(60)
        
        manager1.update(UI_REFRESH_RATE)    #manager의 변경된 사항을 기록한 채, 업데이트 하기
        manager1.draw_ui(SCREEN)            #화면에 출력

        manager.update(UI_REFRESH_RATE)     #manager1의 변경된 사항을 기록한 채, 업데이트 하기
        manager.draw_ui(SCREEN)             #화면에 출력

        pygame.display.update()

def get_num():      #맨 처음 숫자 받기 출력 함수
    while True:
        UI_REFRESH_RATE = clock.tick(60)/1000   #프로그램의 fps를 초당 최대 60로 설정
        for event in pygame.event.get():     #게임 실행하는 동안
            if event.type == pygame.QUIT:   #프로그램을 종료하는 이벤트 발생 시
                pygame.quit()
                sys.exit()
            if (event.type == pygame_gui.UI_TEXT_ENTRY_FINISHED and
                event.ui_object_id == '#main_text_line'):  
                #'#main_text_line'라는 오브젝트 이름을 가진 텍스트 박스(text_input)에 entry입력이 확인된다면
                game_num(event.text)        #입력된 숫자를 갖고 game_num()함수로 이동
            
            manager.process_events(event)   #text_input에 기록된 숫자(이벤트)를 이벤트 변수라고 저장
        
        SCREEN.blit(img,(0,0))  #img변수를 (0, 0)좌표 화면에 출력

        manager.update(UI_REFRESH_RATE)     #manager의 변경된 사항을 기록한 채, 업데이트 하기
        manager.draw_ui(SCREEN)             #화면에 출력

        pygame.display.update()

def game_num(event):    #숫자 비교 and 게임 메인
    global conut, s, b, Out, event_1        #전역변수 conut, s, b, Out, event_1 호출
    num_list = list(map(int, str(event)))   #입력된 event숫자를 각각 순서대로 num_list에 한글자씩 저장
    slist = [event_1, s, b, Out]    #볼카운트용 리스트 선언, 이전에 내가 입력한 값과 결과를 저장함
    s = b = Out = 0     #s,b,Out 초기화

    for i in range(4):  #4번 반복
        if alist[i] == num_list[i]: #컴퓨터의 숫자와 내가 입력한 숫자and 위치 비교
            s += 1
        elif num_list[i] in alist:  #컴퓨터의 숫자 안에 내가 입력한 숫자가 있는지 비교
            b += 1
        else:       #숫자도 없을 경우
            Out += 1

    event_1 = event     
    #event_1은 로그를 위해 만든 변수, event값을 저장함

    if s == 4:  #4 스트라이크 일 때
        end_game_win()  #end_game_win()함수 호출
    else:
        if conut > 0:
            text_log.append_html_text("<br>  {} | {} | {} | {}     ".format(slist[0], slist[1], slist[2], slist[3]))
            #text_log에 "..." html_text를 추가
        show_num(event)     #show_num()함수에 event변수를 넣어 호출
    
def end_game_lose():        #게임 실패 화면
    while True:
        for event in pygame.event.get():    #게임을 실행하는 동안
            if event.type == pygame.QUIT:   #프로그램을 종료하는 이벤트가 일어날 때
                pygame.quit()
                sys.exit()
    
        SCREEN.fill("white")        #화면은 하얗게 칠하기

        #화면에 텍스트 출력하기
        lose_game = pygame.font.SysFont("bahnschrift", 130).render(f"You Lose", True, "red")
        lose_game_rect = lose_game.get_rect(center=(WIDTH/2, HEIGHT/2))
        SCREEN.blit(lose_game,lose_game_rect)

        pygame.display.update()
        
def end_game_win():     #게임 성공 화면
    while True:
        for event in pygame.event.get():    #게임을 실행하는 동안
            if event.type == pygame.QUIT:   #프로그램을 종료하는 이벤트가 일어날 때
                pygame.quit()
                sys.exit()
    
        SCREEN.fill("white")        #화면은 하얗게 칠하기

        #화면에 텍스트 출력하기
        win_game = pygame.font.SysFont("bahnschrift", 130).render(f"You Win", True, "blue")
        win_game_rect = win_game.get_rect(center=(WIDTH/2, HEIGHT/2))
        SCREEN.blit(win_game,win_game_rect)

        #내가 남은 목숨을 화면에 출력하기
        win_conut = pygame.font.SysFont("malgungothic", 25).render(f"남은 목숨 : {10-conut}", True, "blue")
        win_conut_rect = win_conut.get_rect(center=(WIDTH/2, HEIGHT/2+80))
        SCREEN.blit(win_conut,win_conut_rect)

        pygame.display.update()

get_num()   #get_num()함수 불러오기