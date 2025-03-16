import streamlit as st
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

def calculate_1rm(weight, reps):
    return weight * (1 + (0.0333 * reps) + (36 / (37 - reps)) + reps ** 0.1) / 3

def calculate_1rm_bodyweight(weight, bw, reps):
    return ((weight + bw) * (1 + (0.0333 * reps) + (36 / (37 - reps)) + reps ** 0.1) / 3) - bw

def calculate_1rm_muscle_up(weight, bw, reps):
    return 0.5 * (- (weight ** 2 - 294 * weight - 24 * bw * ((reps ** 1.5) - 1) ** 0.5 + 21600) ** 0.5 + weight + 147)

def calculate_working_weights(one_rm):
    return {
        "80%": round(one_rm * 0.8, 1),
        "85%": round(one_rm * 0.85, 1),
        "90%": round(one_rm * 0.9, 1),
        "95%": round(one_rm * 0.95, 1),
        "100%": round(one_rm, 1),
    }

st.title("Калькулятор прогрессии силы")

exercise = st.selectbox("Выберите упражнение", ["Подтягивания", "Брусья", "Приседания", "Выходы силой"])
weight = st.number_input("Вес отягощения (или тело в кг)", min_value=0.0, step=0.5)
reps = st.number_input("Максимальное число повторений", min_value=1, step=1)
bw = st.number_input("Вес тела (кг)", min_value=0.0, step=0.5)

if st.button("Рассчитать 1ПМ и рабочие веса"):
    if exercise in ["Подтягивания", "Брусья"]:
        one_rm = calculate_1rm_bodyweight(weight, bw, reps)
    elif exercise == "Приседания":
        one_rm = calculate_1rm(weight, reps)
    else:
        one_rm = calculate_1rm_muscle_up(weight, bw, reps)
    
    st.write(f"**Ваш расчетный 1ПМ:** {one_rm:.1f} кг")
    working_weights = calculate_working_weights(one_rm)
    st.write("### Рабочие веса:")
    st.json(working_weights)

    # График прогрессии
    weeks = np.arange(1, 11)
    progress = one_rm * (1 + 0.02 * weeks)  # Простая модель роста 1ПМ на 2% в неделю
    df = pd.DataFrame({"Неделя": weeks, "1ПМ (кг)": progress})

    st.write("### Прогресс 1ПМ по неделям")
    fig, ax = plt.subplots()
    ax.plot(df["Неделя"], df["1ПМ (кг)"], marker="o", linestyle="-")
    ax.set_xlabel("Неделя")
    ax.set_ylabel("1ПМ (кг)")
    ax.grid()
    st.pyplot(fig)
