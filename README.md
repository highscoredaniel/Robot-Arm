# Robot Arm
# 20191126 Lee Dongju
# Last updated: 20231107

import pygame
import numpy as np
import random
import os

pygame.init()
pygame.mixer.init()

current_path = os.path.dirname(__file__)
assets_path = os.path.join(current_path, 'assets')
sound = pygame.mixer.Sound(os.path.join(assets_path, 'arm.wav'))
sound2 = pygame.mixer.Sound(os.path.join(assets_path, 'wing.wav'))
sound3 = pygame.mixer.Sound(os.path.join(assets_path, 'wheel.wav'))
pygame.display.set_caption("20191126 이동주")

FPS = 60   # frames per second
RED = (255, 0, 0)

WINDOW_WIDTH = 1400
WINDOW_HEIGHT = 800



def CirclesOverlap (c1, c2):
    dist12 = np.sqrt( (c1.position[0] - c2.position[0])**2 + (c1.position[1] - c2.position[1])**2 )
    if dist12 < c1.radius + c2.radius:
        return True # if overlaps
    return False
#

class MyCircle():
    def __init__(self, x, y, vx, vy, radius=40, color=None, sound = None):
        self.radius = radius 
        self.x = x
        self.y = y
        self.vx = vx
        self.vy = vy 
        self.ax = 0
        self.ay = .1
        self.org_color = color if color is not None else (100, 200, 255)
        self.color = self.org_color
        self.sound = sound
        return
    
    def update(self):
        
        self.x = self.x + self.vx 
        if self.x + self.radius >= WINDOW_WIDTH:
            self.vx = -1. * self.vx 
        if self.x - self.radius < 0:
            self.vx = -1. * self.vx 

        self.y = self.y + self.vy
        self.vy = self.vy + self.ay 
        if self.y < 0:
            self.vy = 0
            self.y = self.radius + 10
        if self.y + self.radius >= WINDOW_HEIGHT:
            self.vy = -1 * self.vy 
            if self.sound is not None:
                # self.sound.play()
                pass
        return
    
    def draw(self, screen):
        pygame.draw.circle(screen, self.color, 
                           [self.x, self.y], 
                           radius = self.radius, )

    def print(self,):
        print("my class self.number = ", self.number)
        return
# 

def getRegularPolygon(nV, radius=1.):
    angle_step = 360. / nV 
    half_angle = angle_step / 2.

    vertices = []
    for k in range(nV):
        degree = angle_step * k 
        radian = np.deg2rad(degree + half_angle)
        x = radius * np.cos(radian)
        y = radius * np.sin(radian)
        vertices.append( [x, y] )
    #
    print("list:", vertices)

    vertices = np.array(vertices)
    print('np.arr:', vertices)
    return vertices
#

class myTriangle():
    def __init__(self, radius=50, color=(100,0,0), vel=[5.,0]):
        self.radius = radius
        self.vertices = getRegularPolygon(3, radius=self.radius)

        self.color = color

        self.angle = 0.
        self.angvel = np.random.normal(5., 7)

        self.position = np.array([0.,0]) #
        # self.position = self.vertices.sum(axis=0) # 2d array
        self.vel = np.array(vel)
        self.tick = 0

    def update(self,):
        self.tick += 1
        self.angle += self.angvel
        self.position += self.vel

        if self.position[0] >= WINDOW_WIDTH:
            self.vel[0] = -1. * self.vel[0]

        if self.position[0] < 0:
            self.vel[0] *= -1.

        if self.position[1] >= WINDOW_HEIGHT:
            self.vel[1] *= -1.

        if self.position[1] < 0:
            self.vel[1] *= -1

        # print(self.tick, self.position)

        return

    def draw(self, screen):
        R = Rmat(self.angle)
        points = self.vertices @ R.T + self.position
        pygame.draw.polygon(screen, self.color, points)
#

class myPolygon():
    def __init__(self, nvertices = 3, radius=70, color=(100,0,0), vel=[5.,0]):
        self.radius = radius
        self.nvertices = nvertices
        self.vertices = getRegularPolygon(self.nvertices, radius=self.radius)

        self.color = color
        self.color_org = color 

        self.angle = 0.
        self.angvel = np.random.normal(5., 7)

        self.position = np.array([0.,0]) #
        # self.position = self.vertices.sum(axis=0) # 2d array
        self.vel = np.array(vel)
        self.tick = 0

    def update(self,):
        self.tick += 1
        self.angle += self.angvel
        self.position += self.vel

        if self.position[0] >= WINDOW_WIDTH:
            self.vel[0] = -1. * self.vel[0]

        if self.position[0] < 0:
            self.vel[0] *= -1.

        if self.position[1] >= WINDOW_HEIGHT:
            self.vel[1] *= -1.

        if self.position[1] < 0:
            self.vel[1] *= -1

        # print(self.tick, self.position)

        return

    def draw(self, screen):
        R = Rmat(self.angle)
        points = self.vertices @ R.T + self.position

        pygame.draw.polygon(screen, self.color, points)
#


def Rmat(degree):
    rad = np.deg2rad(degree) 
    c = np.cos(rad)
    s = np.sin(rad)
    R = np.array([ [c, -s, 0],
                   [s,  c, 0], [0,0,1]])
    return R

def Tmat(tx, ty):
    Translation = np.array( [
        [1, 0, tx],
        [0, 1, ty],
        [0, 0, 1]
    ])
    return Translation
#

def draw(P, H, screen, color=(100, 200, 200)):
    R = H[:2,:2]
    T = H[:2, 2]
    Ptransformed = P @ R.T + T 
    pygame.draw.polygon(screen, color=color, points=Ptransformed, width=3)
    return
#

#class Wings:
    #

def drawWings(screen, position, theta):
    H0 = Tmat(position[0], position[1])
    x = H0[0,2]
    y = H0[1,2]
    pygame.draw.circle(screen, (255,0,0), (x,y), radius = 2)

    dist = 30
        
    nWings = 500
    delta_degrees = 360 / nWings
    for i in range(nWings):
        H = H0 @ Rmat(theta + delta_degrees * i) @ Tmat(dist,0) @ Tmat(0, -h/2)
        draw(X, H, screen, (0, 0, 255))

    return

def main():
    pygame.init() # initialize the engine


    screen = pygame.display.set_mode( (WINDOW_WIDTH, WINDOW_HEIGHT) )
    clock = pygame.time.Clock()
    
    xVel = 0
    yVel = 0
    jaVel1 = 0
    jaVel2 = 0
    jaVel3 = 0
    rSpeed = 8
    rotSpeed = 8
    #cAngleVel = 0
    w1 = 80
    h1 = 30
    w2 = 40
    h2 = 8
    px = WINDOW_WIDTH / 2 - w1/2
    py = WINDOW_HEIGHT - 10
    X1 = np.array([ [0,0], [w1, 0], [w1, h1], [0, h1] ])
    X2 = np.array([ [0,0], [w2, 0], [w2, h2], [0, h2] ])
    position = [px, py]
    theta = 0
    jointangle1 = 0
    jointangle2 = 0
    jointangle3 = 0
    jointangle4 = 90
    jointangle5 = -90
    jointangle6 = -90

    tick = 0

    done = False
    while not done:
        #  input handling
        tick +=2
        
        
        for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    done = True
                # when key pressed
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        xVel = -rSpeed
                        sound3.play()
                    elif event.key == pygame.K_RIGHT:
                        xVel = rSpeed
                        sound3.play()
                    elif event.key == pygame.K_UP:
                        yVel = -rSpeed
                        sound3.play()
                    elif event.key == pygame.K_DOWN:
                        yVel = rSpeed 
                        sound3.play()
                    elif event.key == pygame.K_SPACE:
                        jointangle5 = -120
                        jointangle6 = -60
                        sound.set_volume(0.5)
                        sound.play()
                    elif event.key == pygame.K_z:
                        jaVel1 = -rotSpeed
                        sound2.play()
                    elif event.key == pygame.K_x:
                        jaVel1 = rotSpeed
                        sound2.play()
                    elif event.key == pygame.K_a:
                        jaVel2 = -rotSpeed
                        sound2.play()
                    elif event.key == pygame.K_s:
                        jaVel2 = rotSpeed
                        sound2.play()
                    elif event.key == pygame.K_q:
                        jaVel3 = -rotSpeed
                        sound2.play()
                    elif event.key == pygame.K_w:
                        jaVel3 = rotSpeed
                        sound2.play()
                    

                        #if jointangle5 >= -130 or jointangle6 <= -50:
                            #cAngleVel = 8
                        #elif jointangle5 < -130 or jointangle6 > -50:
                            #cAngleVel = 0
                # when key released
                elif event.type == pygame.KEYUP:
                    if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                        xVel = 0
                        sound3.stop()
                    elif event.key == pygame.K_UP or event.key == pygame.K_DOWN:
                        yVel = 0   
                        sound3.stop()
                    elif event.key == pygame.K_SPACE:
                        jointangle5 = -90
                        jointangle6 = -90
                    elif event.key == pygame.K_z:
                        jaVel1 = 0
                    elif event.key == pygame.K_x:
                        jaVel1 = 0
                    elif event.key == pygame.K_a:
                        jaVel2 = 0
                    elif event.key == pygame.K_s:
                        jaVel2 = 0
                    elif event.key == pygame.K_q:
                        jaVel3 = 0
                    elif event.key == pygame.K_w:
                        jaVel3 = 0

        #movement
        position[0] += xVel
        position[1] += yVel
        jointangle1 += jaVel1
        jointangle2 += jaVel2
        jointangle3 += jaVel3

        print(jointangle1)
    

        
        
        theta += random.randrange(1,10)

        # drawing
        screen.fill((0, 0, 0))

        #initial position
        pygame.draw.circle(screen, (255,0,0), position, radius = 3)
        
        

        #base
        H0 = Tmat(position[0], position[1]) @ Tmat(0, -h1)
        draw(X1, H0, screen, (0,255,0))
       
        # first_arm
        H1 = H0 @ Tmat(w1/2,0) 
        x, y = H1[0,2], H1[1,2] #joint1
        H11 = H1 @ Rmat(-90) @ Tmat(0, -h1/2)
        pygame.draw.circle(screen, (255,0,0), (x,y), radius = 3)
        #jointangle1 = wiggleCount * np.sin(np.deg2rad(tick))
        # draw(X, H11, screen, (0,255,0))
        H12 = H11 @ Tmat(0,h1/2) @ Rmat(jointangle1) @ Tmat(0, -h1/2)
        draw(X1, H12, screen, (0,255,0))

        # second_arm
        H2 = H12 @ Tmat(w1,0) @ Tmat(0, h1/2) #joint2
        x, y = H2[0,2], H2[1,2]
        pygame.draw.circle(screen, (255,0,0), (x,y), radius = 3)
        #jointangle2 = wiggleCount * np.sin(np.deg2rad(tick)) #wiggle effect
        H21 = H2 @ Rmat(jointangle2) @ Tmat(0, -h1/2)
        draw(X1, H21, screen, (0,255,0))

        # third_arm
        H3 = H21 @ Tmat(w1,0) @ Tmat(0, h1/2) #joint3
        x, y = H3[0,2], H3[1,2]
        pygame.draw.circle(screen, (255,0,0), (x,y), radius = 3)
        #jointangle3 = wiggleCount * np.sin(np.deg2rad(tick)) #wiggle effect
        H31 = H3 @ Rmat(jointangle3) @ Tmat(0, -h1/2)
        draw(X1, H31, screen, (0,255,0))

        # claw_base
        H4 = H31 @ Tmat(w1,0) @ Tmat(0, h1/2) #joint4
        x, y = H4[0,2], H4[1,2]
        pygame.draw.circle(screen, (0,255,0), (x,y), radius = 3)
        H41 = H4 @ Rmat(jointangle4) @ Tmat(-w2/2, -h2)
        draw(X2, H41, screen, (0,255,0))

        # right_claw
        H5 = H41 @ Tmat(w2,0) @ Tmat(0, h2/2)
        x, y = H5[0,2], H5[1,2]
        pygame.draw.circle(screen, (255,0,0), (x,y), radius = 3)
        H51 = H5 @ Rmat(jointangle5) @ Tmat(0, -h2/2)
        draw(X2, H51, screen, (0,255,0))

        # left_claw
        H6 = H41 @ Tmat(0, h2/2)
        x, y = H6[0,2], H6[1,2]
        pygame.draw.circle(screen, (255,0,0), (x,y), radius = 3)
        H61 = H6 @ Rmat(jointangle6) @ Tmat(0, -h2/2)
        draw(X2, H61, screen, (0,255,0))


       






            
        # pygame.draw.circle(screen, RED, (cx, cy), radius=3)
        # finish
        pygame.display.flip()
        clock.tick(FPS)
    # end of while
# end of main()

if __name__ == "__main__":
    main()
