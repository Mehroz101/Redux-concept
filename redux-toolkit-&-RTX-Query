# Mastering Redux Toolkit + RTK Query in Next.js 🚀

This README is your go-to documentation for building scalable, maintainable, and efficient data flows in Next.js using Redux Toolkit, RTK Query, and Axios with full token-based authentication.

---

## 📌 PHASE 1: Redux Basics 🔁

### Core Concepts:

* `createSlice` – For modular state logic
* `configureStore` – For setting up the Redux store
* `useSelector`, `useDispatch` – For accessing state and dispatching actions

### Code Example:

```ts
// features/counter/counterSlice.ts
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```

```ts
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counter/counterSlice';

export const store = configureStore({
  reducer: { counter: counterReducer },
});
```

```ts
// _app.tsx
import { Provider } from 'react-redux';
import { store } from '../store';

function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <Component {...pageProps} />
    </Provider>
  );
}
```

```ts
// Usage in component
import { useSelector, useDispatch } from 'react-redux';
import { increment } from '@/features/counter/counterSlice';

const Counter = () => {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return <button onClick={() => dispatch(increment())}>Increment ({count})</button>;
};
```

---

## 📌 PHASE 2: RTK Query 🔄

### Core Concepts:

* `createApi`
* `builder.query()` and `builder.mutation()`
* `tagTypes`, `providesTags`, `invalidatesTags`

### Code Example:

```ts
// services/blogApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const blogApi = createApi({
  reducerPath: 'blogApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Blog'],
  endpoints: (builder) => ({
    getBlogs: builder.query({
      query: () => '/blogs',
      providesTags: ['Blog'],
    }),
    addBlog: builder.mutation({
      query: (newBlog) => ({
        url: '/blogs',
        method: 'POST',
        body: newBlog,
      }),
      invalidatesTags: ['Blog'],
    }),
  }),
});

export const { useGetBlogsQuery, useAddBlogMutation } = blogApi;
```

```ts
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import { blogApi } from './services/blogApi';

export const store = configureStore({
  reducer: {
    [blogApi.reducerPath]: blogApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(blogApi.middleware),
});
```

```ts
// Component.tsx
const BlogList = () => {
  const { data, isLoading } = useGetBlogsQuery();
  const [addBlog] = useAddBlogMutation();

  return (
    <div>
      {data?.map((blog) => <p key={blog.id}>{blog.title}</p>)}
      <button onClick={() => addBlog({ title: 'New Post' })}>Add Blog</button>
    </div>
  );
};
```

---

## 📌 PHASE 3: Axios + Token Logic 🔐

### Core Concepts:

* `axiosInstance` for centralizing API logic
* Request/response interceptors for injecting tokens and handling expiration

### Code Example:

```ts
// utils/axiosInstance.ts
import axios from 'axios';

const axiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

// Request Interceptor
axiosInstance.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response Interceptor
axiosInstance.interceptors.response.use(
  (res) => res,
  async (error) => {
    if (error.response?.status === 401) {
      // TODO: Refresh token logic here
    }
    return Promise.reject(error);
  }
);

export default axiosInstance;
```

---

## 📌 PHASE 4: Integration 🔧

### Core Concepts:

* Replacing `fetchBaseQuery` with custom `axiosBaseQuery`
* Unified token injection and refresh logic

### Code Example:

```ts
// utils/axiosBaseQuery.ts
import type { BaseQueryFn } from '@reduxjs/toolkit/query';
import axios from 'axios';

export const axiosBaseQuery = ({ baseUrl }: { baseUrl: string }): BaseQueryFn => async ({ url, method, data }) => {
  try {
    const result = await axios({ url: baseUrl + url, method, data });
    return { data: result.data };
  } catch (axiosError: any) {
    return { error: axiosError.response?.data || axiosError.message };
  }
};
```

```ts
// services/api.ts
import { createApi } from '@reduxjs/toolkit/query/react';
import { axiosBaseQuery } from '@/utils/axiosBaseQuery';

export const baseApi = createApi({
  reducerPath: 'baseApi',
  baseQuery: axiosBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Blog'],
  endpoints: (builder) => ({
    getBlogs: builder.query({
      query: () => ({ url: '/blogs', method: 'GET' }),
      providesTags: ['Blog'],
    }),
  }),
});
```

---

## 📌 PHASE 5: Advanced Concepts 🚀

### Core Concepts:

* **Optimistic Updates**: Immediately reflect changes in UI
* **Caching & Invalidation**: Invalidate and re-fetch on demand
* **SSR Compatibility**: Hydrate data server-side with `getServerSideProps`
* **Performance**: Prefetching, polling, and cache optimization

### Code Example (Optimistic Update):

```ts
addBlog: builder.mutation({
  query: (blog) => ({
    url: '/blogs',
    method: 'POST',
    data: blog,
  }),
  async onQueryStarted(blog, { dispatch, queryFulfilled }) {
    const patchResult = dispatch(
      baseApi.util.updateQueryData('getBlogs', undefined, (draft) => {
        draft.push(blog);
      })
    );
    try {
      await queryFulfilled;
    } catch {
      patchResult.undo();
    }
  },
}),
```

---

## ✅ Conclusion

This roadmap provides a production-ready setup that brings Redux Toolkit, RTK Query, and Axios together in a scalable architecture. Each phase builds logically toward a robust solution with authentication, dynamic caching, and advanced performance optimization.

Make sure to experiment and adapt these tools based on the specific needs of your application.
