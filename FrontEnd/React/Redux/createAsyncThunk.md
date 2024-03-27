# [Redux]createAsyncThunk?

createAsyncThunk는 Redux Toolkit 에서 제공하는 함수로, 비동기 로직을 처리하는 액션 생성자를 쉽게 만들 수 있게 도와줍니다. 

이 함수를 사용하여 비동기 작업(e.g) 데이터 가져오기, API호출 등) 을 수행하고, 그 결과에 따라 Redux 상태를 자동으로 업데이트 할 수 있습니다.

### Thunk 란 ?  ⇒

특정 작업을 나중에 할 수 있도록 미루기 위해 함수 형태로 감싼 것!

```jsx
// 즉시 실행
const addOne = x => x + 1;
addOne(1);

// 나중에 실행
const addOneThunk = x => () => addOne(x);
setTimeOut(()=>{
  	addOneThunk(1) // 1초 뒤 실행
}, 1000);
```

**redux-thunk & createAsyncThunk**

⇒ Redux 에서 비동기 처리를 위해서는 Redux-thunk 미들웨어를 설치하고 추가해주어야 한다.

⇒ Redux Toolkit 에는 기본적으로 thunk middleware 가 미들웨어에 추가되어있기 때문에 redux-thunk 미들웨어를 설치할 필요가 없다.

⇒ createAsyncThunk API를 통해 fullfilled, rejected, pending 3가지 상태에 대해 reducer 를 작성할 수 있다.

⇒ createAsyncThunk 는 redux-saga 에서만 사용할 수 있던 기능(이미 호출한 API 요청 취소하기 등)도 사용할 수 있다.

### createAsyncThunk 의 기본구조

```jsx
createAsyncThunk(actionType, payloadCreator, options)
```

- actionType : 액션의 타입을 정의하는 문자열입니다. 이 타입은 생성된 액션과 리듀서에서 사용됩니다.
- payloadCreator : 비동기 로직을 실행하는 함수입니다. 이 함수는 비동기 작업을 수행하고, 그 결과를 반환하거나 예외를 던집니다.  PayloadCreator 함수는 두 개의 인자를 받을 수 있습니다.
    - 첫 번째 인자는 액션에 전달되는 payload입니다.
    - 두 번째 인자는 ThunkAPI 객체로, dispatch, getState, extra, rejectWithValue, fulfilled, rejected, pending 등의 필드를 가지고 있습니다.
- options : 액션 생성자의 동작을 커스텀할 수 있는 선택적 매개변수입니다.

### createAsyncThunk 사용 시 생성되는 액션 타입

⇒ createAsyncThunk 를 사용하여 비동기 액션을 생성하면, 해당 비동기 작업의 생명주기에 따라 세 가지 상태의 액션 타입이 자동으로 생성됩니다.

- actionType/pending : 비동기 작업이 시작될 때 디스패치 됩니다.
- actionType/fulfilled : 비동기 작업이 성공적으로 완료되었을 때 디스패치 됩니다.
- actionType/rejected : 비동기 작업이 실패했을 때 디스패치 됩니다.

```jsx
import { createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await axios.get(`/api/users/${userId}`);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

```

이 예제에서는 fetchUserById라는 비동기 액션을 생성합니다. 이 액션은 특정 ID를 가진 사용자의 데이터를 가져오는 API 요청을 수행합니다. 요청이 성공하면 응답 데이터를 반환하고, 실패하면 에러 데이터를 반환합니다.

createAsyncThunk를 사용함으로써, 비동기 작업을 보다 선언적이고 효율적으로 관리할 수 있으며, Redux 상태 업데이트 로직을 간소화할 수 있습니다.

<video src="/img/createAsyncThunk.mov">

참고 : [https://dev.to/ifeanyichima/what-is-createasyncthunk-in-redux--mhe](https://dev.to/ifeanyichima/what-is-createasyncthunk-in-redux--mhe)

참고 : [https://ko.redux.js.org/tutorials/essentials/part-5-async-logic/](https://ko.redux.js.org/tutorials/essentials/part-5-async-logic/)