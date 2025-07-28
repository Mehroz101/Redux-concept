
## 📘 **PHASE 1: Redux Toolkit Basics (`createSlice`, `configureStore`, `useSelector`, `useDispatch`)**

### File: `features/counterSlice.ts`

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1; // directly mutating state with Immer
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

#### 🔍 Line-by-Line Breakdown:

* `createSlice`: Creates a Redux slice with actions and reducer in one place.
* `PayloadAction`: Type helper to define the expected payload type.
* `initialState`: Base state of the counter.
* Reducers:

  * `increment`: Adds `1` to the counter.
  * `decrement`: Subtracts `1`.
  * `incrementByAmount`: Dynamically increases the value by any number.

---

### File: `store.ts`

```ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

#### 🔍 Line-by-Line Breakdown:

* `configureStore`: Initializes the store with middleware & dev tools.
* `RootState`: Infers the state type.
* `AppDispatch`: Typing for dispatching actions.

---

### Usage in Component:

```tsx
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from './features/counterSlice';

const CounterComponent = () => {
  const count = useSelector((state: RootState) => state.counter.value);
  const dispatch = useDispatch<AppDispatch>();

  return (
    <>
      <h1>{count}</h1>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </>
  );
};
```

---

## 📘 **PHASE 2: RTK Query (`createApi`, `builder.query/mutation`, `tagTypes`)**

### File: `services/blogApi.ts`

```ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const blogApi = createApi({
  reducerPath: 'blogApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api/' }),
  tagTypes: ['Blog'],
  endpoints: (builder) => ({
    getBlogs: builder.query<Blog[], void>({
      query: () => 'blogs',
      providesTags: ['Blog'],
    }),
    createBlog: builder.mutation<void, Blog>({
      query: (blog) => ({
        url: 'blogs',
        method: 'POST',
        body: blog,
      }),
      invalidatesTags: ['Blog'],
    }),
  }),
});

export const { useGetBlogsQuery, useCreateBlogMutation } = blogApi;
```

#### 🔍 Line-by-Line Breakdown:

* `createApi`: Initializes RTK Query API service.
* `reducerPath`: Unique name for reducer slice.
* `tagTypes`: Enables caching and invalidation.
* `getBlogs`: Query to fetch data.
* `createBlog`: Mutation to add blog & invalidate cache.
* Hooks like `useGetBlogsQuery()` are auto-generated.

---

### Store Integration:

```ts
import { blogApi } from './services/blogApi';

export const store = configureStore({
  reducer: {
    [blogApi.reducerPath]: blogApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(blogApi.middleware),
});
```

---

## 📘 **PHASE 3: Axios + Token Logic (`axiosInstance`, `interceptors`, `refresh`)**

### File: `utils/axiosInstance.ts`

```ts
import axios from 'axios';

const axiosInstance = axios.create({
  baseURL: 'https://api.example.com',
});

axiosInstance.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

axiosInstance.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response.status === 401) {
      // Logic to refresh token and retry
    }
    return Promise.reject(error);
  }
);

export default axiosInstance;
```

#### 🔍 Breakdown:

* `axios.create`: Configures global instance.
* Request Interceptor: Adds token to header.
* Response Interceptor: Handles token expiry and retry logic.

---

## 📘 **PHASE 4: Integration (`baseApi`, `axiosBaseQuery`, real data flow)**

### File: `services/baseQuery.ts`

```ts
import axiosInstance from '../utils/axiosInstance';

export const axiosBaseQuery =
  ({ baseUrl = '' } = {}) =>
  async ({ url, method, data, params }) => {
    try {
      const result = await axiosInstance({
        url: baseUrl + url,
        method,
        data,
        params,
      });
      return { data: result.data };
    } catch (axiosError) {
      return {
        error: {
          status: axiosError.response?.status,
          data: axiosError.response?.data,
        },
      };
    }
  };
```

---

### Updated `blogApi.ts` to Use Axios:

```ts
import { createApi } from '@reduxjs/toolkit/query/react';
import { axiosBaseQuery } from './baseQuery';

export const blogApi = createApi({
  baseQuery: axiosBaseQuery({ baseUrl: '/api/' }),
  endpoints: (builder) => ({
    getBlogs: builder.query<Blog[], void>({
      query: () => ({ url: 'blogs', method: 'GET' }),
    }),
    createBlog: builder.mutation<void, Blog>({
      query: (blog) => ({
        url: 'blogs',
        method: 'POST',
        data: blog,
      }),
    }),
  }),
});
```

---

## 📘 **PHASE 5: Advanced Concepts (`Optimistic Updates`, `Caching`, `SSR`, `Performance`)**

### Optimistic Updates

```ts
createBlog: builder.mutation({
  query: (blog) => ({
    url: 'blogs',
    method: 'POST',
    data: blog,
  }),
  async onQueryStarted(blog, { dispatch, queryFulfilled }) {
    const patchResult = dispatch(
      blogApi.util.updateQueryData('getBlogs', undefined, (draft) => {
        draft.push(blog);
      })
    );
    try {
      await queryFulfilled;
    } catch {
      patchResult.undo(); // rollback if failed
    }
  },
});
```

---

### SSR Support in Next.js

Use `HYDRATE` from `next-redux-wrapper` and `getServerSideProps()` to prefetch data server-side. You’ll use the `initStore` pattern and `makeStore` from Redux Toolkit to configure hydration.

---

### Caching

RTK Query handles caching automatically using `tagTypes` and `providesTags`/`invalidatesTags`. Keep your endpoints lean and focused on consistent tags to maintain cache integrity.

---

### Performance Tips

* Use `selectFromResult` to minimize rerenders.
* Memoize large components with `React.memo`.
* Only invalidate tags when necessary.

