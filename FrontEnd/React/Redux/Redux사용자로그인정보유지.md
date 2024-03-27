# 사용자의 로그인정보 유지 Redux

### Redux Toolkit(@reduxjs/toolkit)

Redux Toolkit 은 Redux 어플리케이션 개발을 더 쉽게 만들기 위한 도구 모음입니다.
기존 Redux의 많은 부분을 간소화하고, 보일러플레이트 코드를 줄이며, 좋은 관행을 적용할 수 있게 해줍니다.

- CreateSlice : 상태의 일부분을 관리하는 Reducer와 action 을 쉽게 정의할 수 있게 해줍니다. 
name, 초기 상태(initialState), 리듀서함수, 츄가 리듀서(extraReducers) 를 설정할 수 있습니다.
- createAsyncThunk : 비동기 로직을 처리하는 액션을 생성합니다. 첫 번째 파라미터는 액션 타입의 문자열이고, 2번째 파라미터는 비동기 로직을 실행하는 함수입니다.

### redux-persist

redux-persist 는 Redux 스토어의 상태를 브라우저의 로컬 스토리지나 세션 스토리지에 영구적으로 저장할 수 있게 해주는 라이브러리 입니다. 이렇게 하면 페이지를 새로 고침하거나 앱을 다시 시작해도 상태가 유지됩니다.

- persistReducer : 리듀서를 감싸서, 해당 리듀서의 상태를 자동으로 저장하고 복원하는 기능을 추가합니다
- persistStore : 스토어를 감싸서, 저장된 상태를 복원하고, 앞으로의 상태변화를 저장합니다.

### 작동 원리

1. 로그인 과정:
    1. 사용자가 로그인 양식을 제출하면, login이라는 createAsyncThunk 액션이 호출됩니다.
    2. login 액션은 비동기 요청을 보내 사용자 인증을 시도합니다. 성공하면 응답으로 받은 토큰과 사용자 정보를 반환합니다.
    3. authSlice의 extraReducers에서 login.fulfilled를 처리하여, 스토어의 인증 상태(isAuthenticated, token, user)를 업데이트합니다.
2. 상태 저장 및 복원
    1. redux-persist 는 auth 리듀서의 상태를 로컬 스토리지에 자동으로 저장합니다.(whitelist 에 auth 가 포함되어있기 때문)
    2. 사용자가 페이지를 새로고침하거나 앱을 다시 시작하면, redux-persist 는 저장된 상태를 복원하여 auth 리듀서의 상태를 이전 세션의 상태로 설정합니다
3. 로그아웃 과정 
    1. 사용자가 로그아웃 요청하면, authSlice의 logout 액션이 호출됩니다.
    2. logout 액션은 인증 상태를 초기화합니다.(isAuthenticated, token, user를 초기값으로 설정).

이 과정을 통해 어플리케이션은 사용자 인증 상태를 효과적으로 관리하며, 세션이 유지되는 동안 사용자가 로그인 상태를 유지할 수 있습니다. Redux Toolkit과 redux-persist를 함께 사용함으로써, 개발자는 상태 관리 로직을 보다 쉽고 효율적으로 구현할 수 있습니다.

```jsx
// src/redux/slice/authSlice.js
import { createSlice } from '@reduxjs/toolkit';
import { login } from '../authAction'; // login thunk를 임포트합니다.

아래 initialState 는 슬라이스의 초기 상태를 정의합니다
const initialState = {
  isAuthenticated: false,
  token: null,
  user: null,
  loading: false,
  error: null,
};

아래 createSlice 함수는 슬라이스를 생성하며 이 함수는 슬라이스의 이름 , 초기상태, 리듀서, 추가리듀서를 설정합니다
const authSlice = createSlice({
  name: 'auth',
  initialState,

  reducers 객체는 액션 타입에 따라 상태를 어떻게 변경할지 정의합니다.
	loginStart: 로그인 프로세스가 시작되면 loading 상태를 true로 설정합니다.
	loginSuccess: 로그인이 성공하면, 인증 상태, 토큰, 사용자 정보를 업데이트하고, 로딩 상태와 에러를 초기화합니다.
	loginFailure: 로그인이 실패하면, 로딩 상태를 false로 설정하고, 에러 메시지를 저장합니다.
	logout: 로그아웃을 처리하며, 인증 상태, 토큰, 사용자 정보를 초기화합니다
  reducers: {
    loginStart: (state) => {
      state.loading = true;
    },
    loginSuccess: (state, action) => {
      state.isAuthenticated = true;
      state.token = action.payload;
      state.user = action.payload.user;
      state.loading = false;
      state.error = null;
    },
    loginFailure: (state, action) => {
      state.loading = false;
      state.error = action.payload;
    },
    logout: (state) => {
      state.isAuthenticated = false;
      state.token = null;
      state.user = null;
    },
  },
  
  extraReducers는 외부 액션(여기서는 login thunk 액션)에 반응하여 상태를 업데이트합니다. 
  login.fulfilled는 login 액션이 성공적으로 완료되었을 때 호출됩니다. 
  이 때, 인증 상태를 true로 설정하고, 토큰과 사용자 정보를 업데이트합니다.
  extraReducers: (builder) => {
    builder.addCase(login.fulfilled, (state, action) => {
      state.isAuthenticated = true;
      state.token = action.payload.token;
      state.user = action.payload.user;
    });
  },
});

export const { loginStart, loginSuccess, loginFailure, logout } = authSlice.actions;

export default authSlice.reducer;

```

**authAction.js** 파일은 Redux Toolkit 의 **createAsyncThunk** 를 사용하여 비동기 액션을 생성합니다. 저의 경우 사용자 로그인과정을 Login 비동기 액션을 정의하고있습니다.

```jsx
// src/redux/authAction.js
import { createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';
import sessionStorage from 'redux-persist/es/storage/session';

// reactWithValue : 액션이 실패했을 때, 에러 값을 커스텀하게 처리하기 위한 함수입니다.
export const login = createAsyncThunk("user/login", async (loginData, { rejectWithValue }) => {
  try {
    const response = await axios.post("http://localhost:8081/api/v1/users/login", loginData,{
      headers: {
        'Content-Type' : 'application/json'
      }
    });
    console.log(response);
    const { token, user } = response.data.result;
    sessionStorage.setItem('token', token);
    return { token, user };
  } catch (error) {
    console.error(error);
    return rejectWithValue(error.response.data);
  }
});

```