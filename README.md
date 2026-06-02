import streamlit as style
import streamlit as st
from google import genai
from google.genai import types
from google.genai.errors import APIError

# 1. 페이지 설정 및 제목
st.set_page_config(page_title="내 미래를 청해봄 🌱", page_icon="🎓", layout="centered")
st.title("🌱 내 미래를 청해봄: 진로 고민 상담소")
st.caption("진로, 진학, 직업에 대한 고민을 편하게 이야기해보세요. Gemini가 함께 고민해 드립니다.")

# 2. Streamlit Secrets에서 API 키 불러오기 및 클라이언트 초기화
if "GEMINI_API_KEY" not in st.secrets:
    st.error("Streamlit Secrets에 'GEMINI_API_KEY'가 설정되지 않았습니다. 대시보드 설정을 확인해주세요.")
    st.stop()

# 정식 google-genai SDK 클라이언트 생성
client = genai.Client(api_key=st.secrets["GEMINI_API_KEY"])

# 3. 세션 상태(Session State)로 채팅 기록 초기화
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": "안녕하세요! 저는 여러분의 진로 고민을 함께 나눌 AI 상담사입니다. 어떤 분야에 관심이 있나요? 혹은 요즘 어떤 고민을 하고 계시는지 편하게 말씀해 주세요. 😊"
        }
    ]

# 4. 기존 채팅 기록 화면에 표시
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# 5. 사용자 입력 받기
if user_input := st.chat_input("진로에 대한 고민을 적어보세요... (예: 개발자가 되고 싶은데 무엇부터 해야 할까요?)"):
    
    # 사용자 메시지 화면에 표시 및 세션 저장
    with st.chat_message("user"):
        st.markdown(user_input)
    st.session_state.messages.append({"role": "user", "content": user_input})

    # AI 응답 생성 구역
    with st.chat_message("assistant"):
        message_placeholder = st.empty()
        message_placeholder.markdown("💡 *생각 중...*")
        
        try:
            # Gemini에 전달할 대화 기록 구성
            # (system_instruction을 사용하여 학생 상담원 페르소나 부여)
            system_instruction = """
            당신은 중·고등학생 및 대학생을 대상으로 하는 따뜻하고 전문적인 진로 상담 선생님입니다.
            - 친절하고, 공감하며, 격려하는 어조를 사용하세요.
            - 학생이 현실적인 조언과 실천 방안을 얻을 수 있도록 구체적인 단계를 제안해 주세요.
            - 부담을 주기보다는 학생의 강점을 찾아주는 방향으로 대화하세요.
            - 답변은 가독성이 좋게 이모지와 줄바꿈을 적절히 사용해 주세요.
            """
            
            # 이전 대화 맥락을 API 형식에 맞게 변환
            contents = []
            for msg in st.session_state.messages:
                # API 구조에 맞게 user 또는 model로 역할 매핑
                role = "user" if msg["role"] == "user" else "model"
                contents.append(types.Content(
                    role=role,
                    parts=[types.Part.from_text(text=msg["content"])]
                ))
            
            # API 호출 (gemini-2.5-flash-lite 사용)
            response = client.models.generate_content(
                model='gemini-2.5-flash-lite',
                contents=contents,
                config=types.GenerateContentConfig(
                    system_instruction=system_instruction,
                    temperature=0.7,
                )
            )
            
            # 결과 출력 및 저장
            ai_response = response.text
            message_placeholder.markdown(ai_response)
            st.session_state.messages.append({"role": "assistant", "content": ai_response})
            
        except APIError as e:
            # Gemini API 관련 오류 처리
            error_msg = f"⚠️ API 오류가 발생했습니다: {e.message}"
            message_placeholder.markdown(error_msg)
        except Exception as e:
            # 기타 일반 오류 처리
            error_msg = f"⚠️ 죄송합니다. 응답을 생성하는 중 예상치 못한 오류가 발생했습니다. ({str(e)})"
            message_placeholder.markdown(error_msg)
