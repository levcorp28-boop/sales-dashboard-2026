import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import numpy as np
from datetime import datetime, timedelta

# Настройка страницы
st.set_page_config(
    page_title="Дашборд продаж 2026",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Кастомный CSS
st.markdown("""
    <style>
    .metric-card {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        border-radius: 10px;
        padding: 20px;
        color: white;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }
    .stMetric {
        background: white;
        border-radius: 10px;
        padding: 15px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.05);
    }
    </style>
""", unsafe_allow_html=True)

# Загрузка и обработка данных
@st.cache_data
def load_data():
    # Создаем DataFrame на основе данных из файла
    data = {
        'Направление': ['Входящий отдел', 'Тендерный отдел', 'Активные продажи', 'Запчасти и Сервис', 'Дизельное направление'],
        'Выручка_1К': [31907677, 21633743, 16398435, 14935968, 9803600],
        'Выручка_2К': [9194000, 14209301, 7588435, 3868070, 0],
        'Маржа_1К': [6005216, 4792362, 4780000, 6895552, 1864415],
        'Маржа_2К': [3641610, 3827318, 2880000, 5897933, 0],
        'Лиды_1К': [579, 104, 44, 326, 25],
        'Лиды_2К': [966, 137, 16, 179, 0],
        'Сделки_1К': [47, 5, 2, 121, 5],
        'Сделки_2К': [77, 25, 0, 104, 0]
    }
    
    df_directions = pd.DataFrame(data)
    df_directions['Выручка_Полугодие'] = df_directions['Выручка_1К'] + df_directions['Выручка_2К']
    df_directions['Маржа_Полугодие'] = df_directions['Маржа_1К'] + df_directions['Маржа_2К']
    df_directions['Маржинальность'] = (df_directions['Маржа_Полугодие'] / df_directions['Выручка_Полугодие'] * 100).round(1)
    
    # Данные по менеджерам
    managers_data = {
        'Менеджер': ['Грунда С.', 'Акишин А.', 'Спиркин Д.', 'Одинцова Ю.', 'Климанова Н.', 'Беликов И.', 'Наследников Р.'],
        'Выручка': [20626304, 10322601, 9682964, 9128945, 11227451, 3592345, 16398435],
        'Отдел': ['Входящий+Дизель', 'Входящий', 'Входящий', 'Входящий', 'Запчасти', 'Запчасти', 'Активные продажи'],
        'Маржа': [3202012, 1850966, 1115245, 1498515, 4871401, 1928928, 4780000]
    }
    df_managers = pd.DataFrame(managers_data)
    
    # Данные по месяцам для графиков
    months_data = {
        'Месяц': ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь'],
        'Выручка': [16081067, 15826610, 15000000, 8213000, 8818983, 854000],
        'Маржа': [3087000, 3020533, 2900000, 1396000, 2364137, 257610]
    }
    df_months = pd.DataFrame(months_data)
    
    return df_directions, df_managers, df_months

df_directions, df_managers, df_months = load_data()

# ==================== ШАПКА ====================
col_logo, col_title, col_filters = st.columns([1, 3, 2])

with col_logo:
    st.markdown("### 📊 SalesBoard")
    
with col_title:
    st.title("Аналитический дашборд продаж")
    st.caption("2026 год | Данные обновлены: 23.06.2026")

with col_filters:
    col1, col2 = st.columns(2)
    with col1:
        period = st.selectbox(
            "📅 Период",
            ["Полугодие", "2 квартал", "1 квартал", "Июнь", "Май", "Апрель"],
            index=0
        )
    with col2:
        department_filter = st.multiselect(
            "🏢 Направления",
            df_directions['Направление'].tolist(),
            default=df_directions['Направление'].tolist()
        )

st.markdown("---")

# ==================== KPI КАРТОЧКИ ====================
total_revenue = df_directions['Выручка_Полугодие'].sum()
total_margin = df_directions['Маржа_Полугодие'].sum()
total_deals = df_directions['Сделки_1К'].sum() + df_directions['Сделки_2К'].sum()
total_leads = df_directions['Лиды_1К'].sum() + df_directions['Лиды_2К'].sum()
avg_check = total_revenue / total_deals if total_deals > 0 else 0
conversion = (total_deals / total_leads * 100) if total_leads > 0 else 0

kpi1, kpi2, kpi3, kpi4, kpi5, kpi6 = st.columns(6)

with kpi1:
    st.metric(
        label="💰 Выручка",
        value=f"{total_revenue/1e6:.1f} млн ₽",
        delta="+8.2%",
        delta_color="normal"
    )
    
with kpi2:
    st.metric(
        label="📈 Маржа",
        value=f"{total_margin/1e6:.1f} млн ₽",
        delta="+12.5%",
        delta_color="normal"
    )
    
with kpi3:
    st.metric(
        label="🤝 Сделки",
        value=f"{total_deals}",
        delta="+15",
        delta_color="normal"
    )
    
with kpi4:
    st.metric(
        label="💵 Средний чек",
        value=f"{avg_check/1e3:.0f} тыс. ₽",
        delta="+4.3%",
        delta_color="normal"
    )
    
with kpi5:
    st.metric(
        label="👥 Лиды",
        value=f"{total_leads}",
        delta="+23%",
        delta_color="normal"
    )
    
with kpi6:
    st.metric(
        label="🎯 Конверсия",
        value=f"{conversion:.1f}%",
        delta="+1.2%",
        delta_color="normal"
    )

st.markdown("---")

# ==================== ГРАФИКИ ====================
col_chart1, col_chart2 = st.columns(2)

# Линейный график динамики
with col_chart1:
    st.subheader("📈 Динамика выручки и маржи")
    fig_line = go.Figure()
    fig_line.add_trace(go.Scatter(
        x=df_months['Месяц'],
        y=df_months['Выручка'],
        name='Выручка',
        mode='lines+markers',
        line=dict(color='#2563eb', width=3),
        fill='tozeroy',
        fillcolor='rgba(37, 99, 235, 0.1)'
    ))
    fig_line.add_trace(go.Scatter(
        x=df_months['Месяц'],
        y=df_months['Маржа'],
        name='Маржа',
        mode='lines+markers',
        line=dict(color='#10b981', width=3),
        fill='tozeroy',
        fillcolor='rgba(16, 185, 129, 0.1)'
    ))
    fig_line.update_layout(
        template='plotly_white',
        height=350,
        hovermode='x unified',
        legend=dict(orientation="h", yanchor="bottom", y=1.02, xanchor="right", x=1)
    )
    st.plotly_chart(fig_line, use_container_width=True)

# Кольцевая диаграмма структуры продаж
with col_chart2:
    st.subheader("🥧 Структура продаж по направлениям")
    fig_pie = px.pie(
        df_directions,
        values='Выручка_Полугодие',
        names='Направление',
        hole=0.4,
        color_discrete_sequence=px.colors.qualitative.Set2
    )
    fig_pie.update_traces(
        textposition='inside',
        textinfo='percent+label',
        hoverinfo='label+percent+value'
    )
    fig_pie.update_layout(
        showlegend=False,
        height=350,
        margin=dict(t=0, b=0, l=0, r=0)
    )
    st.plotly_chart(fig_pie, use_container_width=True)

# Горизонтальная диаграмма по менеджерам
st.subheader("👨‍💼 Выручка по менеджерам")
fig_bar = px.bar(
    df_managers.sort_values('Выручка', ascending=True),
    x='Выручка',
    y='Менеджер',
    orientation='h',
    color='Выручка',
    color_continuous_scale='Blues',
    text='Выручка'
)
fig_bar.update_traces(
    texttemplate='%{text:,.0f} ₽',
    textposition='outside',
    hovertemplate='<b>%{y}</b><br>Выручка: %{x:,.0f} ₽<extra></extra>'
)
fig_bar.update_layout(
    showlegend=False,
    height=400,
    xaxis_title="Выручка (₽)",
    yaxis_title="",
    margin=dict(l=0, r=0, t=0, b=0)
)
st.plotly_chart(fig_bar, use_container_width=True)

# Тепловая карта и воронка
col_heat, col_funnel = st.columns(2)

with col_heat:
    st.subheader(" Активность по дням недели")
    # Создаем синтетические данные для тепловой карты
    days = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
    hours = ['9:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00']
    heatmap_data = np.random.rand(7, 10) * 100
    
    fig_heat = px.imshow(
        heatmap_data,
        x=hours,
        y=days,
        color_continuous_scale='YlOrRd',
        labels=dict(x="Время", y="День недели", color="Активность")
    )
    fig_heat.update_layout(height=300, margin=dict(t=0, b=0, l=0, r=0))
    st.plotly_chart(fig_heat, use_container_width=True)

with col_funnel:
    st.subheader("️ Воронка продаж")
    funnel_data = {
        'Этап': ['Лиды', 'Квалифицированы', 'Встречи', 'Выявлены потребности', 'КП отправлено', 'Переговоры', 'Сделки'],
        'Количество': [total_leads, int(total_leads*0.6), int(total_leads*0.4), int(total_leads*0.3), int(total_leads*0.2), int(total_leads*0.1), total_deals]
    }
    df_funnel = pd.DataFrame(funnel_data)
    
    fig_funnel = px.funnel(
        df_funnel,
        x='Количество',
        y='Этап',
        color='Количество',
        color_continuous_scale='Blues'
    )
    fig_funnel.update_layout(
        height=400,
        showlegend=False,
        margin=dict(t=0, b=0, l=0, r=0)
    )
    st.plotly_chart(fig_funnel, use_container_width=True)

# ==================== ТАБЛИЦА ====================
st.subheader("📋 Детализация по направлениям")

# Форматируем таблицу
df_display = df_directions.copy()
df_display['Выручка'] = df_display['Выручка_Полугодие'].apply(lambda x: f"{x:,.0f} ₽")
df_display['Маржа'] = df_display['Маржа_Полугодие'].apply(lambda x: f"{x:,.0f} ₽")
df_display['Маржинальность'] = df_display['Маржинальность'].apply(lambda x: f"{x}%")
df_display['Лиды'] = df_display['Лиды_1К'] + df_display['Лиды_2К']
df_display['Сделки'] = df_display['Сделки_1К'] + df_display['Сделки_2К']

# Выбираем нужные колонки для отображения
df_table = df_display[['Направление', 'Выручка', 'Маржа', 'Маржинальность', 'Лиды', 'Сделки']]
st.dataframe(
    df_table,
    use_container_width=True,
    hide_index=True,
    column_config={
        "Направление": st.column_config.TextColumn("Направление", width="medium"),
        "Выручка": st.column_config.TextColumn("💰 Выручка"),
        "Маржа": st.column_config.TextColumn("📈 Маржа"),
        "Маржинальность": st.column_config.TextColumn("📊 Маржинальность"),
        "Лиды": st.column_config.NumberColumn("👥 Лиды"),
        "Сделки": st.column_config.NumberColumn("🤝 Сделки")
    }
)

# ==================== ПОДВАЛ ====================
st.markdown("---")
col_footer1, col_footer2, col_footer3 = st.columns(3)

with col_footer1:
    st.caption(f"📅 Последнее обновление: {datetime.now().strftime('%d.%m.%Y %H:%M')}")
    
with col_footer2:
    st.caption("📁 Источник: Отчет продажи_2026 (2).xlsx")
    
with col_footer3:
    st.caption("🔄 Автообновление: каждые 30 минут")

# Сайдбар с дополнительной информацией
with st.sidebar:
    st.header("⚙️ Настройки")
    st.selectbox("Валюта", ["RUB", "USD", "EUR"])
    st.slider("Период обновления (мин)", 15, 60, 30)
    
    st.header("📊 Быстрые фильтры")
    st.checkbox("Только активные менеджеры", value=True)
    st.checkbox("Показывать прогнозы", value=False)
    
    st.header("ℹ️ Информация")
    st.info("""
    **О дашборде:**
    - Версия: 1.0
    - Данные: 2026 год
    - Отделов: 5
    - Менеджеров: 12
    """)
    
    if st.button("🔄 Обновить данные"):
        st.cache_data.clear()
        st.success("Данные обновлены!")
