---
layout : page
title : Python/Pygame 3D ASCII Spinning Earth with Moon
---
### Original Youtube Video : Part 1 - https://www.youtube.com/watch?v=7Q6yvpjvKVg&t Part 2 - https://www.youtube.com/watch?v=zxszimDGJHA <br> Thank you to Giovanni Code for creating this amazing project :D <br> Codes are below
~~~ python
import pygame as pg
import numpy as np
from math import pi, sin, cos

# Initialize Pygame and define constants
pg.init()
clock = pg.time.Clock()
FPS = 30

WIDTH = 1920
HEIGHT = 1080

R = 300
R_moon = R / 3.7
MAP_WIDTH = 139
MAP_HEIGHT = 34

MAP_WIDTH_MOON = 49
MAP_HEIGHT_MOON = 15

black = (0, 0, 0)
green = (0, 255, 0)
blue = (0, 0, 255)
grey = (100, 100, 100)
dark_grey = (50, 50, 50)

# Load fonts
my_font = pg.font.SysFont('arial', 20)
my_font_moon = pg.font.SysFont('arial', 12)

# Load and process the ASCII maps
with open('C:\\Users\\hwseo\\Downloads\\earth_2_0_W140_H35 (1).txt', 'r') as file:
    data = [file.read().replace('\n', '')]

ascii_chars = [char for line in data for char in line]
inverted_ascii_chars = ascii_chars[::-1]

with open('C:\\Users\\hwseo\\Downloads\\moon_W50_H16 (1).txt', 'r') as file:
    data = [file.read().replace('\n', '')]

moon_ascii_chars = [char for line in data for char in line]
inverted_moon_ascii_chars = moon_ascii_chars[::-1]

class Projection:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.screen = pg.display.set_mode((width, height))
        self.background = black
        pg.display.set_caption('ASCII 3D EARTH')
        self.surfaces = {}

    def addSurface(self, name, surface):
        self.surfaces[name] = surface

    def display(self):
        self.screen.fill(self.background)
        for key, value in self.surfaces.items():
            if key == 'globe':
                for i, node in enumerate(value.nodes):
                    text = inverted_ascii_chars[i]
                    color = green if text == '+' else blue
                    text_surface = my_font.render(text, False, color)
                    text_surface_dark = my_font.render(text, False, grey if text == '+' else dark_grey)
                    if node[0] > 0 and node[1] > 0:
                        self.screen.blit(text_surface, (WIDTH / 2 + int(node[0]), HEIGHT / 2 + int(node[2])))
                    elif node[0] < 0 and node[1] > 0:
                        self.screen.blit(text_surface_dark, (WIDTH / 2 + int(node[0]), HEIGHT / 2 + int(node[2])))
            elif key == 'moon':
                center = value.findCentre()
                for j, node in enumerate(value.nodes):
                    text = inverted_moon_ascii_chars[j]
                    text_surface = my_font_moon.render(text, False, grey)
                    text_surface_dark = my_font_moon.render(text, False, dark_grey)
                    if (node[0] > center[0] and node[0] > R and node[1] > -R / 3 * 2) or \
                       (node[0] > center[0] and node[0] < -R and node[1] > -R / 3 * 2) or \
                       (node[0] > center[0] and node[0] > -R and node[0] < R and node[1] > 0):
                        self.screen.blit(text_surface, (WIDTH / 2 + int(node[0]), HEIGHT / 2 + int(node[2])))
                    elif (node[0] < center[0] and node[0] > R and node[1] > -R / 3 * 2) or \
                         (node[0] < center[0] and node[0] < -R and node[1] > -R / 3 * 2) or \
                         (node[0] < center[0] and node[0] > -R and node[0] < R and node[1] > 0):
                        self.screen.blit(text_surface_dark, (WIDTH / 2 + int(node[0]), HEIGHT / 2 + int(node[2])))

    def moveAll(self, theta):
        for key, value in self.surfaces.items():
            center = value.findCentre()
            c, s = np.cos(theta), np.sin(theta)
            rotate_matrix = np.array([[c, -s, 0, 0],
                                      [s, c, 0, 0],
                                      [0, 0, 1, 0],
                                      [0, 0, 0, 1]])
            value.rotate(center, rotate_matrix)
            if key == 'moon':
                dx, dy = R * 2 * np.cos(theta), R * 2 * np.sin(theta)
                transform_matrix = np.array([[1, 0, 0, 0],
                                             [0, 1, 0, 0],
                                             [0, 0, 1, 0],
                                             [dx, dy, 0, 1]])
                scale_c = 0.5
                scale_d = 0.25
                sx = sy = sz = scale_c + scale_d if dy != 0 else scale_c + scale_d
                scale_matrix = np.array([[sx, 0, 0, 0],
                                         [0, sy, 0, 0],
                                         [0, 0, sz, 0],
                                         [0, 0, 0, 1]])
                value.transform(transform_matrix)
                value.scale(center, scale_matrix)

class Object:
    def __init__(self):
        self.nodes = np.zeros((0, 4))

    def addNodes(self, node_array):
        ones_column = np.ones((len(node_array), 1))
        self.nodes = np.hstack((node_array, ones_column))

    def findCentre(self):
        return self.nodes.mean(axis=0)

    def rotate(self, center, matrix):
        self.nodes = center + np.dot(self.nodes - center, matrix)

    def transform(self, matrix):
        self.nodes = np.dot(self.nodes, matrix)

    def scale(self, center, matrix):
        self.nodes = center + np.dot(self.nodes - center, matrix)

def generate_sphere_nodes(radius, map_width, map_height):
    nodes = []
    for i in range(map_height + 1):
        lat = (pi / map_height) * i
        for j in range(map_width + 1):
            lon = (2 * pi / map_width) * j
            x = round(radius * sin(lat) * cos(lon), 2)
            y = round(radius * sin(lat) * sin(lon), 2)
            z = round(radius * cos(lat), 2)
            nodes.append((x, y, z))
    return nodes

# Generate nodes for Earth and Moon
earth_nodes = generate_sphere_nodes(R, MAP_WIDTH, MAP_HEIGHT)
moon_nodes = generate_sphere_nodes(R_moon, MAP_WIDTH_MOON, MAP_HEIGHT_MOON)

spin = 0
running = True

while running:
    clock.tick(FPS)
    pv = Projection(WIDTH, HEIGHT)

    earth = Object()
    moon = Object()
    earth.addNodes(np.array(earth_nodes))
    moon.addNodes(np.array(moon_nodes))
    pv.addSurface('globe', earth)
    pv.addSurface('moon', moon)
    pv.moveAll(spin)
    pv.display()

    for event in pg.event.get():
        if event.type == pg.QUIT:
            running = False

    pg.display.update()
    clock.tick(149587234895708942357823490572348907590238759)
    spin -= 0.03

pg.quit()
~~~
