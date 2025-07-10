# Biceps-Brachii-Simulation
This is simulation of Biceps Brachii muscle using physics to explain how the arm work. Hopefully this will help you understand how this muscle works better. Try it!

import tkinter as tk
from tkinter import ttk
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from matplotlib.patches import Rectangle
import numpy as np

def calculate_force(P, R, s):
    return 0.5 * P * R * s + np.pi * P * R**2

def update_plot(*args):
    P = float(P_var.get())
    s = float(s_var.get())
    R0 = float(R0_var.get())
    ks = float(ks_var.get())

    R = R0 * (1 - ks * (s / max_s))
    R = max(R, 0.001)
    mg = calculate_force(P, R, s)

    result_label.config(text=f"Force (mg) = {mg:.2f} N\nRadius (R) = {R:.3f} m")

    ax.clear()
    ax.set_xlim(-1, 1)
    ax.set_ylim(0, 2.2)
    ax.set_aspect('equal')
    ax.axis('off')
    ax.set_title("Biceps Brachii Simulation")

    shoulder = np.array([0, 1.5])
    upper_length = 0.6
    forearm_length = 0.6

    elbow = shoulder + np.array([0, -upper_length])

    # ปรับมุม ให้ s มาก แขนจะเหยียด
    angle_deg = 90 + (s / max_s) * 90
    angle_rad = np.radians(angle_deg)
    forearm = elbow + forearm_length * np.array([np.cos(angle_rad), np.sin(angle_rad)])

    # Arm
    ax.plot([shoulder[0], elbow[0]], [shoulder[1], elbow[1]], lw=6, color='brown')  # upper arm
    ax.plot([elbow[0], forearm[0]], [elbow[1], forearm[1]], lw=6, color='orange')  # forearm

    # Muscle
    muscle_y = (shoulder[1] + elbow[1]) / 2
    muscle_width = R * 2
    ellipse = plt.Circle((0, muscle_y), muscle_width, color='red', alpha=0.6)
    ax.add_patch(ellipse)

    weight_length = 0.3
    weight_top = forearm
    weight_bottom = forearm + np.array([0, -weight_length])
    ax.plot([weight_top[0], weight_bottom[0]],
            [weight_top[1], weight_bottom[1]],
            color='black', linestyle='--', lw=1)
    weight_width = 0.25
    weight_height = 0.25
    ax.add_patch(Rectangle(
        (weight_bottom[0] - weight_width / 2, weight_bottom[1] - weight_height / 2),
        weight_width, weight_height, color='gray'))

    ax.text(weight_bottom[0] + 0.05, weight_bottom[1] - 0.02, "mg", fontsize=10, color='black')

    canvas.draw()

max_s = 0.3

root = tk.Tk()
root.title("Biceps Brachii Simulation")

mainframe = ttk.Frame(root, padding="10")
mainframe.grid(row=0, column=0)

P_var = tk.DoubleVar(value=50)
s_var = tk.DoubleVar(value=0.1)
R0_var = tk.DoubleVar(value=0.05)
ks_var = tk.DoubleVar(value=0.5)

ttk.Label(mainframe, text="Pressure (P) [kPa]").grid(row=0, column=0, sticky=tk.W)
ttk.Scale(mainframe, variable=P_var, from_=60, to=220, orient="horizontal", command=update_plot).grid(row=0, column=1)

ttk.Label(mainframe, text="Contraction Distance (s) [m]").grid(row=1, column=0, sticky=tk.W)
ttk.Scale(mainframe, variable=s_var, from_=0.01, to=max_s, orient="horizontal", command=update_plot).grid(row=1, column=1)

ttk.Label(mainframe, text="Initial Radius (R0) [m]").grid(row=2, column=0, sticky=tk.W)
ttk.Scale(mainframe, variable=R0_var, from_=0.1, to=0.2, orient="horizontal", command=update_plot).grid(row=2, column=1)

ttk.Label(mainframe, text="Rate of R (ks)").grid(row=3, column=0, sticky=tk.W)
ttk.Scale(mainframe, variable=ks_var, from_=0.0, to=1.0, orient="horizontal", command=update_plot).grid(row=3, column=1)

result_label = ttk.Label(mainframe, text="", font=("Arial", 12))
result_label.grid(row=4, column=0, columnspan=2, pady=10)

fig, ax = plt.subplots(figsize=(4.5, 4))
canvas = FigureCanvasTkAgg(fig, master=mainframe)
canvas.get_tk_widget().grid(row=5, column=0, columnspan=2)

update_plot()
root.mainloop()
