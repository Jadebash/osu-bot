import re
import time
import pyautogui
import pygetwindow as gw
import numpy as np
from typing import List

class CurvePointsS:
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y

class TimingPoint:
    def __init__(self, offset: int, real_mpb: float):
        self.offset = offset
        self.real_mpb = real_mpb

class DifficultyS:
    def __init__(self, slider_multiplier: float):
        self.slider_multiplier = slider_multiplier

class HitObject:
    def __init__(self, hit_object_line: str, timing_points: List[TimingPoint], timing_point_index: int, difficulty: DifficultyS):
        self.process_hit_object_line(hit_object_line, timing_points, timing_point_index, difficulty)

    def process_hit_object_line(self, hit_object_line: str, timing_points: List[TimingPoint], timing_point_index: int, difficulty: DifficultyS):
        hit_object_line = re.sub(r'\s+', '', hit_object_line)
        elements = hit_object_line.split(',')
        
        self.x = int(elements[0])
        self.y = int(elements[1])
        self.time = int(elements[2])
        type_val = int(elements[3])
        
        if type_val & 1:
            self.type = "circle"
        elif type_val & 2:
            self.type = "slider"
            self.repeat = int(elements[6])
            self.pixel_length = float(elements[7])
            real_current_mpb = self.get_real_current_mpb(self.time, timing_points, timing_point_index)
            self.slider_duration = self.pixel_length / (100.0 * difficulty.slider_multiplier) * real_current_mpb * self.repeat
        elif type_val & 8:
            self.type = "spinner"
            self.spinner_end_time = int(elements[5])
    
    def get_real_current_mpb(self, hit_object_time: int, timing_points: List[TimingPoint], timing_point_index: int) -> float:
        if timing_point_index == len(timing_points) - 1:
            return timing_points[timing_point_index].real_mpb
        
        if (hit_object_time >= timing_points[timing_point_index].offset and 
            hit_object_time < timing_points[timing_point_index + 1].offset) or \
           (hit_object_time < timing_points[timing_point_index].offset and timing_point_index == 0):
            return timing_points[timing_point_index].real_mpb
        
        counter = 1
        while (timing_point_index + counter < len(timing_points) - 1 and 
               timing_points[timing_point_index + 1 + counter].offset <= hit_object_time):
            counter += 1
        
        timing_point_index += counter
        return timing_points[timing_point_index].real_mpb

def get_osu_window():
    try:
        window = gw.getWindowsWithTitle("osu!")[0]
        return {"top": window.top, "left": window.left, "width": window.width, "height": window.height}
    except IndexError:
        print("Osu! window not found.")
        return None

def parse_osu_file(osu_file_path, difficulty_multiplier):
    with open(osu_file_path, "r", encoding="utf-8") as file:
        lines = file.readlines()
    
    hit_objects = []
    timing_points = []
    parsing_hit_objects = False
    parsing_timing_points = False

    for line in lines:
        line = line.strip()

        if line.startswith("[TimingPoints]"):
            parsing_timing_points = True
            continue
        if line.startswith("[HitObjects]"):
            parsing_hit_objects = True
            parsing_timing_points = False
            continue

        if parsing_timing_points:
            elements = line.split(',')
            if len(elements) >= 2:
                try:
                    offset = int(float(elements[0]))
                    real_mpb = float(elements[1])
                    timing_points.append(TimingPoint(offset, real_mpb))
                except ValueError:
                    continue

        if parsing_hit_objects:
            hit_objects.append(line)
    
    if not timing_points:
        print("Warning: No valid timing points found in the .osu file.")
        timing_points.append(TimingPoint(0, 500))

    difficulty = DifficultyS(slider_multiplier=difficulty_multiplier)
    parsed_objects = [HitObject(obj, timing_points, 0, difficulty) for obj in hit_objects]
    return parsed_objects

def play_hit_objects(hit_objects, osu_region):
    print("Waiting 4 seconds before starting...")  
    time.sleep(4)  

    for obj in hit_objects:
        x_pos = osu_region["left"] + obj.x + osu_region["width"] // 2 - 400  #dumb shit is always on the top left
        y_pos = osu_region["top"] + obj.y + osu_region["height"] // 2 - 300  
        
        if obj.type == "circle":
            pyautogui.click(x_pos, y_pos)
        elif obj.type == "slider":
            pyautogui.mouseDown(x_pos, y_pos)
            slider_duration = max(0, obj.slider_duration / 1000)
            time.sleep(slider_duration)
            pyautogui.mouseUp()
        elif obj.type == "spinner":
            pyautogui.moveTo(x_pos, y_pos)
            for _ in range(10):
                pyautogui.moveRel(30, 30)
                pyautogui.moveRel(-30, -30)
        
        time.sleep(0.1)  

if __name__ == "__main__":
    osu_region = get_osu_window()
    if not osu_region:
        exit()
    
    hit_objects = parse_osu_file("C:\\Users\\your_pcname_here\\AppData\\Local\\osu!\\Songs\\851543 Culprate - Florn\\Culprate - Florn (StarrStyx) [pishi's Easy].osu", 1.4)
    play_hit_objects(hit_objects, osu_region)
