import streamlit as st

st.set_page_config(page_title="진로 고민 상담소", page_icon="🌈")

st.title("🌈 학생 진로 고민 상담소")
st.write("나에게 어울리는 직업을 찾아보자!")

name = st.text_input("이름을 입력하세요")

subject = st.selectbox(
    "좋아하는 과목은?",
    ["국어", "영어", "수학", "미술", "체육", "과학"]
)

personality = st.radio(
    "성격은 어떤 편인가요?",
    ["활발하다", "조용하다", "창의적이다", "꼼꼼하다"]
)

environment = st.selectbox(
    "어떤 환경에서 일하고 싶나요?",
    ["사람들과 함께", "조용한 실내", "아이들과 함께", "자유로운 분위기"]
)

if st.button("추천 받기"):
    
    job = ""
    reason = ""

    if subject == "미술":
        job = "디자이너"
        reason = "창의력을 활용할 수 있어요!"

    elif environment == "아이들과 함께":
        job = "유치원 교사"
        reason = "아이들과 소통하며 즐겁게 일할 수 있어요!"

    elif personality == "꼼꼼하다":
        job = "연구원"
        reason = "세심하게 분석하는 능력이 잘 어울려요!"

    elif personality == "활발하다":
        job = "승무원"
        reason = "사람들과 소통하는 능력을 살릴 수 있어요!"

    else:
        job = "마케팅 기획자"
        reason = "다양한 아이디어를 활용할 수 있어요!"

    st.success(f"{name}님에게 어울리는 직업은 👉 {job}!")
    st.info(reason)

st.write("---")
st.caption("학생들의 진로 고민 해결을 위한 간단한 프로그램")
