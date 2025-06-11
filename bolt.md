## Building an agent app

##### How to do setup:

1. Starting with commands like 

```bash
npx create-next-app@latest 
```

2. give name  -> use EsLint -> Tailwind css - > src/ -> Turbopack 

```bash
npx shadcn@latest init
```

3. To install all elements from shadcn we can 

```bash
npm shadcn@latest add --all
```

4. With the help this we can add components like buttons , calendar , chart , etc.

5. You can do manual changes in the components and it will get relected when we use in our pages.
   
   

## Databases:

1. Add a postgresSQL database (Neon)
   
   1. create a project on neon
   
   2. setup .env file in our local folder and name a variable DATABASE_URL="" copy here key

2. Add drizzle ORM
   
   ```bash
   npm i drizzle-orm @neondatabase/serverless dotenv --legacy-peer-deps
   npm i -D drizzle-kit tsx --legacy-peer-deps
   ```

3. Add schema push schema , verify changes in neon and verify changes in drizzle studio
   
   ```ts
   import {drizzle} from 'drizzle-orm/http';
   const db = drizzle(process.env.DATABASE_URL);
   // this is to initialize the connection
   ```

4. Create , review and merge pull request
   
   ```ts
   //schema.ts
   // this is where we create our schema
   // make sure give proper constraints
   ```

5. create a drizzle.config.ts file:
   
   ```ts
   // drizzle.config.ts
   import "dotenv/config";
   import {defineConfig} from "drizzle-kit";
   
   export default defineConfig({
       out:"./drizzle",
       schema:"./src/db/schema.ts",
       dialect:"postgresql",
       dbCrediantials:{
       url: process.env.DATABASE_URL!,
       },
   })
   ```

6. push to the database
   
   ```bash
   npx drizzle-kit push
   ```

7. it is optional we can change the commands for our ease by changing the package.json.
   
   ```json
   "db-push":"drizzle-kit push",
   "db-studio":"drizzle-kit studio"
   ```

## Authentication Setup

1. Integrate Better-Auth
   BETTER_AUTH_SECRET=""
   BETTER_AUTH_URL="http://localhost:3000  "
   
   ```bash
   npm i better-auth
   ```

2. Configure the authentication methods you want to use. Better Auth comes with built-in support for email/password, and social sign-on providers.
   a.
   
   ```ts
   import {betterAuth} from "better-auth"
   import {drizzleAdapter} from "better-auth/adapters/drizzle"
   import * as schema from "@/db/schema"
   import {db} from "@db"
   
   export const auth = betterAuth({
       emailAndPassword: {  
           enabled: true
       },
       database: drizzleAdapter(db , {provider:"pg",})
   })
   ```
   
   b. this is used to generate the schema for auth: 
   
   ```bash
   npx @better-auth/cli generate 
   ```

3. Push the changes to the drizzle:
   
   ```bash
   npm run db:push
   ```

4. To handle API requests, you need to set up a route handler on your server.
   Create a new file or route in your framework's designated catch-all route handler. This route should handle requests for the path `/api/auth/*` (unless you've configured a different base path).
   
   ```typescript
   /app/api/auth/[...all]/route.ts
   
   import { auth } from "@/lib/auth"; // path to your auth file
   import { toNextJsHandler } from "better-auth/next-js";
   
   export const { POST, GET } = toNextJsHandler(auth);
   ```

5. ### Create Client Instance
   
   The client-side library helps you interact with the auth server. Better Auth comes with a client for all the popular web frameworks, including vanilla JavaScript.
   
   1. Import `createAuthClient` from the package for your framework (e.g., "better-auth/react" for React).
   
   2. Call the function to create your client.
   
   3. Pass the base URL of your auth server. (If the auth server is running on the same domain as your client, you can skip this step.)
      
      ```ts
      lib/auth-client.ts
      
      import { createAuthClient } from "better-auth/react"
      export const authClient = createAuthClient({
          /** The base URL of the server (optional if you're using the same domain) */
          baseURL: "http://localhost:3000"
      })
      ```
      
      

## Sequence Diagram

![13a10b92-41a9-410d-9487-a61164ddc6e3](file:///C:/Users/HP/OneDrive/Pictures/Typedown/13a10b92-41a9-410d-9487-a61164ddc6e3.png)

```tsx
"use client";

import { useState } from "react";
import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";

export default function SignInPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const router = useRouter();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const { data, error } = await authClient.signIn.email(
        {
          email,
          password,
          callbackURL: "/dashboard",
          rememberMe: true,
        },
        {
          onRequest: () => {
            console.log("Signing in...");
          },
          onSuccess: () => {
            console.log("Sign in successful");
            router.push("/dashboard");
          },
          onError: (ctx) => {
            setError(ctx.error.message || "Invalid credentials");
          },
        }
      );
    } catch (err) {
      setError("An unexpected error occurred");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full space-y-8">
        <div>
          <h2 className="mt-6 text-center text-3xl font-extrabold text-gray-900">
            Sign in to your account
          </h2>
        </div>
        <form className="mt-8 space-y-6" onSubmit={handleSubmit}>
          <div className="rounded-md shadow-sm -space-y-px">
            <div>
              <label htmlFor="email" className="sr-only">
                Email address
              </label>
              <input
                id="email"
                name="email"
                type="email"
                autoComplete="email"
                required
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="appearance-none rounded-none relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 rounded-t-md focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 focus:z-10 sm:text-sm"
                placeholder="Email address"
              />
            </div>
            <div>
              <label htmlFor="password" className="sr-only">
                Password
              </label>
              <input
                id="password"
                name="password"
                type="password"
                autoComplete="current-password"
                required
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="appearance-none rounded-none relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 rounded-b-md focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 focus:z-10 sm:text-sm"
                placeholder="Password"
              />
            </div>
          </div>

          {error && (
            <div className="rounded-md bg-red-50 p-4">
              <p className="text-sm text-red-800">{error}</p>
            </div>
          )}

          <div>
            <button
              type="submit"
              disabled={loading}
              className="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {loading ? "Signing in..." : "Sign in"}
            </button>
          </div>

          <div className="flex items-center justify-between">
            <div className="text-sm">
              <a href="/signup" className="font-medium text-indigo-600 hover:text-indigo-500">
                Don't have an account? Sign up
              </a>
            </div>
            <div className="text-sm">
              <a href="/forgot-password" className="font-medium text-indigo-600 hover:text-indigo-500">
                Forgot your password?
              </a>
            </div>
          </div>
        </form>
      </div>
    </div>
  );
}
```

## Form making:

<img title="" src="file:///C:/Users/HP/OneDrive/Pictures/Typedown/4a12992e-86da-47d8-928f-fc6b40a1f4d7.png" alt="4a12992e-86da-47d8-928f-fc6b40a1f4d7" data-align="inline" style="zoom:200%;">

##### Explanation:

Absolutely! Here's a **brief + best example** explanation you can directly use in your notes. I've also added a Romanized Hindi explanation so you remember the concept deeply.

* * *

📘 Notes: Understanding `<Form {...form}>` in React Hook Form
-------------------------------------------------------------

### ✅ Concept:

`<Form {...form}>` is a **custom wrapper component** built using `FormProvider` from **React Hook Form**.

It allows **all nested components** (`FormField`, `Input`, etc.) to **access form context (state, errors, control)** **without manually passing props** like `control`, `errors`, etc.

This helps manage large forms more cleanly and efficiently.

* * *

### 🧠 Romanized Hindi Explanation:

`<Form {...form}>` ek custom wrapper hai jo sare child components ko `useForm()` ka access deta hai automatically. Ab hume `control`, `errors` jaise props har jagah manually pass karne ki zarurat nahi padti.

Yeh context share karta hai poore form ke andar ke har field ke saath.

* * *

### ✅ Best Minimal Example:

#### 1. `Form.tsx` – Custom wrapper using `FormProvider`

```tsx
// components/ui/Form.tsx
import { FormProvider, UseFormReturn } from 'react-hook-form';
import { ReactNode } from 'react';

interface FormProps extends UseFormReturn<any> {
  children: ReactNode;
}

export function Form({ children, ...form }: FormProps) {
  return <FormProvider {...form}>{children}</FormProvider>;
}
```

* * *

#### 2. `LoginForm.tsx` – Using the wrapper

```tsx
import { useForm, useFormContext, Controller } from 'react-hook-form';
import { Form } from './components/ui/Form';

function Input({ label, ...props }: any) {
  return (
    <div>
      <label>{label}</label>
      <input {...props} className="border px-2 py-1" />
    </div>
  );
}

function FormField({ name, label }: { name: string; label: string }) {
  const { control } = useFormContext();

  return (
    <Controller
      name={name}
      control={control}
      render={({ field }) => <Input label={label} {...field} />}
    />
  );
}

export function LoginForm() {
  const form = useForm();

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit((data) => console.log(data))}>
        <FormField name="email" label="Email" />
        <FormField name="password" label="Password" />
        <button type="submit" className="mt-2 bg-blue-500 text-white px-4 py-2">Login</button>
      </form>
    </Form>
  );
}
```

* * *

### ✅ Summary Points:

* `useForm()` creates a form instance.

* `<Form {...form}>` wraps all children using `FormProvider`.

* `useFormContext()` lets nested fields like `FormField` access `form.control`.

* Cleaner and scalable way to build forms without prop-drilling.

* * *

## TRPC

### What is a tRPC Router?

tRPC (TypeScript Remote Procedure Call) is a library that enables end-to-end type-safe APIs between a client (typically a frontend app) and a server (backend). The tRPC router acts as the central hub on the server side, routing incoming requests from the client to the appropriate backend logic or "procedures" (functions) defined in your application. It handles the communication, input validation, and response formatting.

### How tRPC Router Works (High-Level)

1. **Client Side**: The client (e.g., a React app) uses tRPC to make typed API calls as if they were local function calls.
2. **Server Side**: The tRPC router on the server receives these calls, maps them to specific procedures (defined in the router), executes the logic, and returns the result.
3. **Communication**: Under the hood, tRPC uses HTTP (or WebSocket in some cases) to send requests and responses, often leveraging JSON for data transfer.
4. **Type Safety**: Both client and server share TypeScript types, ensuring type safety without manual API contract definitions (like OpenAPI or Swagger).

### Key Details of Each Step

1. **Client Request**: The tRPC client (e.g., in a React app) is auto-generated based on the server's router definition, allowing developers to call server functions as if they were local. For example, `trpc.user.get.query({ id: 1 })` triggers an HTTP request to the server.

2. **Router Parsing**: The tRPC router is a collection of procedures (queries or mutations) defined with a structure like `appRouter.user.get`. It uses the endpoint path (`user.get`) to determine which procedure to invoke.

3. **Input Validation**: Before calling the procedure, the router often validates input using a schema (e.g., Zod), ensuring the data (like `id: 1`) matches expected types.

4. **Procedure Execution**: Procedures are the actual functions where business logic lives. They can interact with databases, external APIs, or other services.

5. **Response**: The router handles errors (e.g., if the procedure throws an exception) and serializes the result into a response format (usually JSON).

6. **Client Update**: The client receives the typed response and can directly use it in the app, benefiting from TypeScript's type safety.
   ![952e6a1c-9de8-4022-bafe-0e823e4d44cf](file:///C:/Users/HP/OneDrive/Pictures/Typedown/952e6a1c-9de8-4022-bafe-0e823e4d44cf.png)

I'll provide a full-fledged example of how a tRPC router works using the latest version of tRPC (as of late 2023, tRPC v10). This example will include a complete setup for both the server and client sides, along with a detailed explanation of each part. I'll also revisit the sequence diagram with more specifics tied to the code. Since I can't render images or execute code, I'll provide the full code in TypeScript/JavaScript and describe the flow.

### Full Example: tRPC Setup for a Simple User Management API

This example builds a small API for managing users, with endpoints for querying a user by ID and creating a new user. We'll use tRPC v10 with a Next.js app (a common setup), but the concepts apply to other frameworks as well.

#### Prerequisites

- Node.js (v16+)
- TypeScript
- Basic knowledge of Next.js or React for the client side
- Dependencies: `@trpc/server`, `@trpc/client`, `@trpc/next`, `zod` (for input validation)

#### Step 1: Project Setup

First, set up a Next.js project with TypeScript if you haven't already:

```bash
npx create-next-app@latest my-trpc-app --typescript --use-npm
cd my-trpc-app
npm install @trpc/server @trpc/client @trpc/next @trpc/react-query zod
```

#### Step 2: Server-Side Setup (tRPC Router)

We'll create the tRPC router, define procedures (API endpoints), and set up the API handler.

1. **Create tRPC Context and Router Files**:
   Create a folder structure like `src/server/trpc` to organize the server code.
   
   - `src/server/trpc/context.ts` (Define context for procedures, e.g., database access or authentication)
     
     ```typescript
     import { inferAsyncReturnType } from '@trpc/server';
     // Context type for tRPC proceduresexport async function createContext() { return {
     
         // Simulate a database or add auth user here
         db: {
           users: [
             { id: 1, name: 'John Doe', email: 'john@example.com' },
           ],
         },
     
     };
     }
     export type Context = inferAsyncReturnType<typeof createContext>;
     ```
   
   - `src/server/trpc/router.ts` (Define the tRPC router with procedures
     
     ```typescript
     import { initTRPC } from '@trpc/server';
     import { z } from 'zod'; import { Context } from './context';
     
     // Initialize tRPC with context 
     const t = initTRPC.context < Context > ().create();
     
     // Export reusable router and procedure helpers 
     export const router = t.router; export const publicProcedure = t.procedure;
     
     // Define the app router with user-related procedures 
     export const appRouter = router({
         user: {
             // Query to get a user by ID 
             getById: publicProcedure.input(z.object({ id: z.number() })).
                 query(({ ctx, input }) => {
                     const user = ctx.db.users.find((u) => u.id === input.id);
                     if (!user) throw new Error('User not found');
                     return user;
                 }),
             // Mutation to create a new user 
             create: publicProcedure.
                 input(z.object({ name: z.string(), email: z.string().email() })).mutation(({ ctx, input }) => {
                     const newUser = { id: ctx.db.users.length + 1, name: input.name, email: input.email, }; ctx.db.users.push(newUser); // Simulate adding to DB
                     return newUser;
                 }),
             },
     
         });
     
     // Export type for client usage
     export type AppRouter = typeof appRouter;
     ```
     
     

2. **Set Up API Handler**:
   Create an API route in Next.js to expose the tRPC router.
- `src/pages/api/trpc/[trpc].ts`
  
  ```typescript
  import { createNextApiHandler } from '@trpc/server/adapters/next'; 
  import { appRouter } from '../../../server/trpc/router'; 
  import { createContext } from '../../../server/trpc/context';
  // Export API handler for tRPC
  export default createNextApiHandler({ 
  router: appRouter, createContext,
  });
  ```
  
  #### Step 3: Client-Side Setup (tRPC Client with React Query)

Now, configure the tRPC client in the Next.js app to communicate with the server.

1. **Create tRPC Client Utility**:
- `src/utils/trpc.ts`
  
  ```typescript
  import { httpBatchLink } from '@trpc/client';
  import { createTRPCReact } from '@trpc/react-query';
  import type { AppRouter } from '../server/trpc/router';
  // Initialize tRPC React client
  export const trpc = createTRPCReact<AppRouter>();
  // Provider configuration for Next.js
  export const TRPCProvider = trpc.Provider;
  // Client configuration (used in _app.tsx)
  export function getTRPCClient() {
  return trpc.createClient({
    links: [
      httpBatchLink({
        url: '/api/trpc', // API endpoint
      }),
    ],
  
  });
  }
  
  ```
2. **Wrap App with tRPC and React Query Providers**:
- `src/pages/_app.tsx`
  
  ```typescript
  import { AppProps } from 'next/app';
  import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
  import { getTRPCClient, TRPCProvider } from '../utils/trpc';
  const queryClient = new QueryClient();
  const trpcClient = getTRPCClient();
  export default function App({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <TRPCProvider client={trpcClient} queryClient={queryClient}>
        <Component {...pageProps} />
      </TRPCProvider>
    </QueryClientProvider>
  );
  }
  
  ```
3. **Use tRPC in a Component**:
- `src/pages/index.tsx`
  
  ```typescript
  import { trpc } from '../utils/trpc';
  export default function Home() {
  // Query to fetch user with ID 1
  const { data: user, isLoading, error } = trpc.user.getById.useQuery({ id: 1 });
  // Mutation to create a new user
  const createUser = trpc.user.create.useMutation();
  const handleCreateUser = async () => {
    try {
      await createUser.mutateAsync({
        name: 'Jane Doe',
        email: 'jane@example.com',
      });
      alert('User created!');
    } catch (err) {
      console.error(err);
      alert('Failed to create user');
    }
  
  };
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return (
    <div>
      <h1>User Data</h1>
      <p>Name: {user?.name}</p>
      <p>Email: {user?.email}</p>
      <button onClick={handleCreateUser}>Create New User</button>
    </div>
  
  );
  }
  
  ```

#### Step 4: Run the Application

- Start the Next.js development server:
  
  ```bash
  npm run dev
  ```

- Open `http://localhost:3000` to see the app. It should display the user data for ID 1 (John Doe) and allow creating a new user via the button.

---

### Detailed Sequence Diagram (Text-Based) for the Example

Using the `getById` query as an example, here’s an updated sequence diagram tied to the code above. I’ll use `->` for messages and include specific references to the code.

```
Participants:
- Client: React Component (src/pages/index.tsx)
- tRPC Client: tRPC React Query Hook (trpc.user.getById.useQuery)
- tRPC Router: Server API Handler (src/pages/api/trpc/[trpc].ts)
- Procedure: Backend Logic (src/server/trpc/router.ts - user.getById)
- Context: Simulated Database (src/server/trpc/context.ts)

Flow:
1. Client -> tRPC Client: Trigger query with trpc.user.getById.useQuery({ id: 1 })
   Note: React component calls the tRPC hook to fetch user data.
2. tRPC Client -> tRPC Router: Send HTTP request to /api/trpc/user.getById with payload { id: 1 }
   Note: tRPC client serializes the request and sends it to the server via HTTP (handled by httpBatchLink).
3. tRPC Router -> tRPC Router: Parse request path "user.getById" and match to procedure
   Note: The router (appRouter) identifies the correct procedure based on the endpoint.
4. tRPC Router -> Context: Create context with createContext()
   Note: Context is initialized, providing access to the simulated DB.
5. tRPC Router -> Procedure: Call user.getById with input { id: 1 } and context
   Note: Router invokes the procedure with validated input (via Zod) and context.
6. Procedure -> Context: Access ctx.db.users to find user with id=1
   Note: Procedure queries the simulated database in the context.
7. Procedure -> tRPC Router: Return user data { id: 1, name: "John Doe", email: "john@example.com" }
   Note: Procedure sends result back to the router.
8. tRPC Router -> tRPC Client: Send HTTP response with JSON data
   Note: Router formats the result as JSON and returns it over HTTP.
9. tRPC Client -> Client: Update React state with user data
   Note: React Query hook updates the component state, triggering a re-render with the fetched data.
10. Client -> Client: Render UI with user details
    Note: Component displays the user’s name and email.
```

---

### Explanation of Key Components (Latest tRPC v10)

1. **Initialization (`initTRPC`)**:
   
   - `initTRPC.context<Context>().create()` sets up the tRPC instance with a typed context, allowing procedures to access shared data like a database or user session.

2. **Procedures**:
   
   - `publicProcedure` defines reusable middleware (none in this example for simplicity) and can be query (read-only) or mutation (write operations).
   - Input validation with `zod` ensures type safety (e.g., `id: z.number()`).

3. **Router Structure**:
   
   - Nested routers (`user` in `appRouter`) allow for organized API endpoints, mimicking REST-like paths (`user.getById`).

4. **Client-Side with React Query**:
   
   - `@trpc/react-query` integrates tRPC with React Query for caching, retries, and state management.
   - Hooks like `useQuery` and `useMutation` make API calls feel like local function calls.

5. **Next.js Integration**:
   
   - `createNextApiHandler` exposes the tRPC router as a Next.js API route, handling HTTP communication.

---

### Key Features of tRPC v10 in This Example

- **Type Safety**: The `AppRouter` type is inferred and shared between client and server, eliminating the need for manual API contracts.
- **Batching**: `httpBatchLink` batches multiple requests into a single HTTP call for efficiency.
- **Error Handling**: Errors (like "User not found") are automatically serialized and sent to the client.
- **Flexibility**: Works with mutations (`create`) and queries (`getById`), plus subscriptions (not shown here).

---

### Example1:

I'll provide a full-fledged example of how a tRPC router works using the latest version of tRPC (as of late 2023, tRPC v10). This example will include a complete setup for both the server and client sides, along with a detailed explanation of each part. I'll also revisit the sequence diagram with more specifics tied to the code. Since I can't render images or execute code, I'll provide the full code in TypeScript/JavaScript and describe the flow.

### Full Example: tRPC Setup for a Simple User Management API

This example builds a small API for managing users, with endpoints for querying a user by ID and creating a new user. We'll use tRPC v10 with a Next.js app (a common setup), but the concepts apply to other frameworks as well.

#### Prerequisites

- Node.js (v16+)
- TypeScript
- Basic knowledge of Next.js or React for the client side
- Dependencies: `@trpc/server`, `@trpc/client`, `@trpc/next`, `zod` (for input validation)

#### Step 1: Project Setup

First, set up a Next.js project with TypeScript if you haven't already:

```bash
npx create-next-app@latest my-trpc-app --typescript --use-npm
cd my-trpc-app
npm install @trpc/server @trpc/client @trpc/next @trpc/react-query zod
```

#### Step 2: Server-Side Setup (tRPC Router)

We'll create the tRPC router, define procedures (API endpoints), and set up the API handler.

1. **Create tRPC Context and Router Files**:
   Create a folder structure like `src/server/trpc` to organize the server code.
   
   - `src/server/trpc/context.ts` (Define context for procedures, e.g., database access or authentication)
     
     ```typescript
     import { inferAsyncReturnType } from '@trpc/server';
     // Context type for tRPC procedures
     export async function createContext() {
     return {
     
       // Simulate a database or add auth user here
       db: {
         users: [
           { id: 1, name: 'John Doe', email: 'john@example.com' },
         ],
       },
     
     };
     }
     export type Context = inferAsyncReturnType<typeof createContext>;
     
     ```
     
     - `src/server/trpc/router.ts` (Define the tRPC router with procedures)
     
     ```typescript
     import { initTRPC } from '@trpc/server';
     import { z } from 'zod';
     import { Context } from './context';
     // Initialize tRPC with context
     const t = initTRPC.context<Context>().create();
     // Export reusable router and procedure helpers
     export const router = t.router;
     export const publicProcedure = t.procedure;
     // Define the app router with user-related procedures
     export const appRouter = router({
     user: {
     
       // Query to get a user by ID
       getById: publicProcedure
         .input(z.object({ id: z.number() }))
         .query(({ ctx, input }) => {
           const user = ctx.db.users.find((u) => u.id === input.id);
           if (!user) throw new Error('User not found');
           return user;
         }),
     
       // Mutation to create a new user
       create: publicProcedure
         .input(z.object({ name: z.string(), email: z.string().email() }))
         .mutation(({ ctx, input }) => {
           const newUser = {
             id: ctx.db.users.length + 1,
             name: input.name,
             email: input.email,
           };
           ctx.db.users.push(newUser); // Simulate adding to DB
           return newUser;
         }),
     
     },
     });
     // Export type for client usage
     export type AppRouter = typeof appRouter;
     ```

2. **Set Up API Handler**:
   Create an API route in Next.js to expose the tRPC router.
   
   - `src/pages/api/trpc/[trpc].ts`
     
     ```typescript
     import { createNextApiHandler } from '@trpc/server/adapters/next';
     import { appRouter } from '../../../server/trpc/router';
     import { createContext } from '../../../server/trpc/context';
     // Export API handler for tRPC
     export default createNextApiHandler({
     router: appRouter,
     createContext,
     });
     ```
     
     #### Step 3: Client-Side Setup (tRPC Client with React Query)

Now, configure the tRPC client in the Next.js app to communicate with the server.

1. **Create tRPC Client Utility**:
   
   - `src/utils/trpc.ts`
     
     ```typescript
     import { httpBatchLink } from '@trpc/client';
     import { createTRPCReact } from '@trpc/react-query';
     import type { AppRouter } from '../server/trpc/router';
     // Initialize tRPC React client
     export const trpc = createTRPCReact<AppRouter>();
     // Provider configuration for Next.js
     export const TRPCProvider = trpc.Provider;
     // Client configuration (used in _app.tsx)
     export function getTRPCClient() {
     return trpc.createClient({
     
       links: [
         httpBatchLink({
           url: '/api/trpc', // API endpoint
         }),
       ],
     
     });
     }
     
     ```

2. **Wrap App with tRPC and React Query Providers**:
   
   - `src/pages/_app.tsx`
     
     ```typescript
     import { AppProps } from 'next/app';
     import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
     import { getTRPCClient, TRPCProvider } from '../utils/trpc';
     const queryClient = new QueryClient();
     const trpcClient = getTRPCClient();
     export default function App({ Component, pageProps }: AppProps) {
     return (
     
       <QueryClientProvider client={queryClient}>
         <TRPCProvider client={trpcClient} queryClient={queryClient}>
           <Component {...pageProps} />
         </TRPCProvider>
       </QueryClientProvider>
     
     );
     }
     
     ```

3. **Use tRPC in a Component**:
   
   - `src/pages/index.tsx`
     
     ```typescript
     import { trpc } from '../utils/trpc';
     export default function Home() {
     // Query to fetch user with ID 1
     const { data: user, isLoading, error } = trpc.user.getById.useQuery({ id: 1 });
     // Mutation to create a new user
     const createUser = trpc.user.create.useMutation();
     const handleCreateUser = async () => {
     
       try {
         await createUser.mutateAsync({
           name: 'Jane Doe',
           email: 'jane@example.com',
         });
         alert('User created!');
       } catch (err) {
         console.error(err);
         alert('Failed to create user');
       }
     
     };
     if (isLoading) return <div>Loading...</div>;
     if (error) return <div>Error: {error.message}</div>;
     return (
     
       <div>
         <h1>User Data</h1>
         <p>Name: {user?.name}</p>
         <p>Email: {user?.email}</p>
         <button onClick={handleCreateUser}>Create New User</button>
       </div>
     
     );
     }
     
     ```

#### Step 4: Run the Application

- Start the Next.js development server:
  
  ```bash
  npm run dev
  ```

- Open `http://localhost:3000` to see the app. It should display the user data for ID 1 (John Doe) and allow creating a new user via the button.

---

### Detailed Sequence Diagram (Text-Based) for the Example

Using the `getById` query as an example, here’s an updated sequence diagram tied to the code above. I’ll use `->` for messages and include specific references to the code.

```
Participants:
- Client: React Component (src/pages/index.tsx)
- tRPC Client: tRPC React Query Hook (trpc.user.getById.useQuery)
- tRPC Router: Server API Handler (src/pages/api/trpc/[trpc].ts)
- Procedure: Backend Logic (src/server/trpc/router.ts - user.getById)
- Context: Simulated Database (src/server/trpc/context.ts)

Flow:
1. Client -> tRPC Client: Trigger query with trpc.user.getById.useQuery({ id: 1 })
   Note: React component calls the tRPC hook to fetch user data.
2. tRPC Client -> tRPC Router: Send HTTP request to /api/trpc/user.getById with payload { id: 1 }
   Note: tRPC client serializes the request and sends it to the server via HTTP (handled by httpBatchLink).
3. tRPC Router -> tRPC Router: Parse request path "user.getById" and match to procedure
   Note: The router (appRouter) identifies the correct procedure based on the endpoint.
4. tRPC Router -> Context: Create context with createContext()
   Note: Context is initialized, providing access to the simulated DB.
5. tRPC Router -> Procedure: Call user.getById with input { id: 1 } and context
   Note: Router invokes the procedure with validated input (via Zod) and context.
6. Procedure -> Context: Access ctx.db.users to find user with id=1
   Note: Procedure queries the simulated database in the context.
7. Procedure -> tRPC Router: Return user data { id: 1, name: "John Doe", email: "john@example.com" }
   Note: Procedure sends result back to the router.
8. tRPC Router -> tRPC Client: Send HTTP response with JSON data
   Note: Router formats the result as JSON and returns it over HTTP.
9. tRPC Client -> Client: Update React state with user data
   Note: React Query hook updates the component state, triggering a re-render with the fetched data.
10. Client -> Client: Render UI with user details
    Note: Component displays the user’s name and email.
```

---

### Explanation of Key Components (Latest tRPC v10)

1. **Initialization (`initTRPC`)**:
   
   - `initTRPC.context<Context>().create()` sets up the tRPC instance with a typed context, allowing procedures to access shared data like a database or user session.

2. **Procedures**:
   
   - `publicProcedure` defines reusable middleware (none in this example for simplicity) and can be query (read-only) or mutation (write operations).
   - Input validation with `zod` ensures type safety (e.g., `id: z.number()`).

3. **Router Structure**:
   
   - Nested routers (`user` in `appRouter`) allow for organized API endpoints, mimicking REST-like paths (`user.getById`).

4. **Client-Side with React Query**:
   
   - `@trpc/react-query` integrates tRPC with React Query for caching, retries, and state management.
   - Hooks like `useQuery` and `useMutation` make API calls feel like local function calls.

5. **Next.js Integration**:
   
   - `createNextApiHandler` exposes the tRPC router as a Next.js API route, handling HTTP communication.

---

### Key Features of tRPC v10 in This Example

- **Type Safety**: The `AppRouter` type is inferred and shared between client and server, eliminating the need for manual API contracts.
- **Batching**: `httpBatchLink` batches multiple requests into a single HTTP call for efficiency.
- **Error Handling**: Errors (like "User not found") are automatically serialized and sent to the client.
- **Flexibility**: Works with mutations (`create`) and queries (`getById`), plus subscriptions (not shown here).

---

## Real World Example:

Let's build a real-world, full-fledged example of a tRPC-based application with practical features. This example will be a **Task Management App** where users can create, update, delete, and list tasks, with authentication for user-specific data. We'll use tRPC v10 (latest as of late 2023), Next.js, Prisma (for a real database), and basic authentication with NextAuth.js. I'll include all necessary code, a realistic setup, and a detailed sequence diagram for a key operation.

### Real-World Example: Task Management App with tRPC

This app will allow authenticated users to manage their tasks. We'll use SQLite with Prisma for simplicity (easily replaceable with PostgreSQL/MySQL), and NextAuth.js for authentication.

#### Goals

- Implement a full-stack app with tRPC for type-safe API communication.
- Use a real database (Prisma + SQLite).
- Include authentication to restrict task access to logged-in users.
- Provide CRUD operations for tasks (Create, Read, Update, Delete).
- Show a detailed sequence diagram for a key operation (e.g., creating a task).

---

### Step 1: Project Setup

Start by setting up a Next.js project with TypeScript and installing the required dependencies.

```bash
npx create-next-app@latest task-manager --typescript --use-npm
cd task-manager
npm install @trpc/server @trpc/client @trpc/next @trpc/react-query zod prisma @prisma/client next-auth
npx prisma init
```

#### Configure Prisma

Edit `prisma/schema.prisma` to define the database schema for users and tasks.

```prisma
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id       String @id @default(uuid())
  email    String @unique
  name     String?
  password String
  tasks    Task[]
}

model Task {
  id          String   @id @default(uuid())
  title       String
  description String?
  completed   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  userId      String
  user        User     @relation(fields: [userId], references: [id])
}
```

Run the following to generate the Prisma client and create the database:

```bash
npx prisma migrate dev --name init
```

---

### Step 2: Authentication Setup with NextAuth.js

Set up NextAuth.js for simple email/password authentication (stored in the database for this example; in production, use bcrypt for passwords).

1. **Create NextAuth Configuration**:
   
   - `src/pages/api/auth/[...nextauth].ts`
     
     ```typescript
     import NextAuth from 'next-auth';
     import CredentialsProvider from 'next-auth/providers/credentials';
     import { PrismaClient } from '@prisma/client';
     const prisma = new PrismaClient();
     export default NextAuth({
     providers: [
     
       CredentialsProvider({
         name: 'Credentials',
         credentials: {
           email: { label: 'Email', type: 'email' },
           password: { label: 'Password', type: 'password' },
         },
         async authorize(credentials) {
           if (!credentials) return null;
           const user = await prisma.user.findUnique({
             where: { email: credentials.email },
           });
           if (user && user.password === credentials.password) {
             return { id: user.id, email: user.email, name: user.name };
           }
           return null;
         },
       }),
     
     ],
     session: {
     
       strategy: 'jwt',
     
     },
     pages: {
     
       signIn: '/login',
     
     },
     });
     ```

2. **Create a Login Page**:
   
   - `src/pages/login.tsx`
     
     ```typescript
     import { useState } from 'react';
     import { signIn } from 'next-auth/react';
     import { useRouter } from 'next/router';
     export default function Login() {
     const [email, setEmail] = useState('');
     const [password, setPassword] = useState('');
     const router = useRouter();
     const handleSubmit = async (e: React.FormEvent) => {
     
       e.preventDefault();
       const result = await signIn('credentials', {
         redirect: false,
         email,
         password,
       });
       if (result?.ok) {
         router.push('/');
       } else {
         alert('Login failed');
       }
     
     };
     return (
     
       <form onSubmit={handleSubmit}>
         <input
           type="email"
           value={email}
           onChange={(e) => setEmail(e.target.value)}
           placeholder="Email"
           required
         />
         <input
           type="password"
           value={password}
           onChange={(e) => setPassword(e.target.value)}
           placeholder="Password"
           required
         />
         <button type="submit">Login</button>
         <p>
           Don't have an account? <a href="/register">Register</a>
         </p>
       </form>
     
     );
     } 
     ```
     
     3. **Create a Registration Page**:
   
   - `src/pages/register.tsx`
     
     ```typescript
     import { useState } from 'react';
     import { useRouter } from 'next/router';
     import axios from 'axios';
     export default function Register() {
     const [email, setEmail] = useState('');
     const [password, setPassword] = useState('');
     const [name, setName] = useState('');
     const router = useRouter();
     const handleSubmit = async (e: React.FormEvent) => {
     
       e.preventDefault();
       try {
         await axios.post('/api/register', { email, password, name });
         router.push('/login');
       } catch (error) {
         alert('Registration failed');
         console.error(error);
       }
     
     };
     return (
     
       <form onSubmit={handleSubmit}>
         <input
           type="text"
           value={name}
           onChange={(e) => setName(e.target.value)}
           placeholder="Name"
         />
         <input
           type="email"
           value={email}
           onChange={(e) => setEmail(e.target.value)}
           placeholder="Email"
           required
         />
         <input
           type="password"
           value={password}
           onChange={(e) => setPassword(e.target.value)}
           placeholder="Password"
           required
         />
         <button type="submit">Register</button>
       </form>
     
     );
     }
     ```

3. **API Route for Registration**:
   
   - `src/pages/api/register.ts`
     
     ```typescript
     import { NextApiRequest, NextApiResponse } from 'next';
     import { PrismaClient } from '@prisma/client';
     const prisma = new PrismaClient();
     export default async function handler(req: NextApiRequest, res: NextApiResponse) {
     if (req.method !== 'POST') {
     
       return res.status(405).json({ error: 'Method not allowed' });
     
     }
     const { email, password, name } = req.body;
     try {
     
       const user = await prisma.user.create({
         data: { email, password, name },
       });
       res.status(201).json({ message: 'User created', userId: user.id });
     
     } catch (error) {
     
       res.status(400).json({ error: 'User already exists or invalid data' });
     
     } finally {
     
       await prisma.$disconnect();
     
     }
     } 
     ```

---

### Step 3: tRPC Server Setup

Set up the tRPC router with context for authentication and database access.

1. **Create tRPC Context**:
   
   - `src/server/trpc/context.ts`
     
     ```typescript
     import { inferAsyncReturnType } from '@trpc/server';
     import { getServerSession } from 'next-auth/next';
     import { PrismaClient } from '@prisma/client';
     import { NextApiRequest } from 'next';
     const prisma = new PrismaClient();
     export async function createContext(opts: { req: NextApiRequest }) {
     const session = await getServerSession(opts);
     return {
     
       prisma,
       session,
       userId: session?.user?.id || null,
     
     };
     }
     export type Context = inferAsyncReturnType<typeof createContext>;
     ```

2. **Create Middleware for Protected Routes**:
   
   - `src/server/trpc/middleware.ts`
     
     ```typescript
     import { TRPCError } from '@trpc/server';
     import { t } from './trpc';
     export const isAuthenticated = t.middleware(({ next, ctx }) => {
     if (!ctx.userId) {
     
       throw new TRPCError({ code: 'UNAUTHORIZED', message: 'Not authenticated' });
     
     }
     return next({ ctx: { ...ctx, userId: ctx.userId } });
     });
     
     ```

3. **Define tRPC Router for Tasks**:
   
   - `src/server/trpc/router.ts`
     
     ```typescript
     import { initTRPC } from '@trpc/server';
     import { z } from 'zod';
     import { Context } from './context';
     import { isAuthenticated } from './middleware';
     const t = initTRPC.context<Context>().create();
     export const router = t.router;
     export const publicProcedure = t.procedure;
     export const protectedProcedure = t.procedure.use(isAuthenticated);
     export const appRouter = router({
     task: {
     
       // Get all tasks for the logged-in user
       getAll: protectedProcedure.query(async ({ ctx }) => {
         return ctx.prisma.task.findMany({
           where: { userId: ctx.userId },
           orderBy: { createdAt: 'desc' },
         });
       }),
     
       // Get a single task by ID
       getById: protectedProcedure
         .input(z.object({ id: z.string() }))
         .query(async ({ ctx, input }) => {
           const task = await ctx.prisma.task.findUnique({
             where: { id: input.id },
           });
           if (!task || task.userId !== ctx.userId) {
             throw new Error('Task not found or unauthorized');
           }
           return task;
         }),
     
       // Create a new task
       create: protectedProcedure
         .input(z.object({ title: z.string(), description: z.string().optional() }))
         .mutation(async ({ ctx, input }) => {
           return ctx.prisma.task.create({
             data: {
               title: input.title,
               description: input.description,
               userId: ctx.userId,
             },
           });
         }),
     
       // Update a task
       update: protectedProcedure
         .input(
           z.object({
             id: z.string(),
             title: z.string().optional(),
             description: z.string().optional(),
             completed: z.boolean().optional(),
           })
         )
         .mutation(async ({ ctx, input }) => {
           const task = await ctx.prisma.task.findUnique({ where: { id: input.id } });
           if (!task || task.userId !== ctx.userId) {
             throw new Error('Task not found or unauthorized');
           }
           return ctx.prisma.task.update({
             where: { id: input.id },
             data: {
               title: input.title,
               description: input.description,
               completed: input.completed,
             },
           });
         }),
     
       // Delete a task
       delete: protectedProcedure
         .input(z.object({ id: z.string() }))
         .mutation(async ({ ctx, input }) => {
           const task = await ctx.prisma.task.findUnique({ where: { id: input.id } });
           if (!task || task.userId !== ctx.userId) {
             throw new Error('Task not found or unauthorized');
           }
           return ctx.prisma.task.delete({ where: { id: input.id } });
         }),
     
     },
     });
     export type AppRouter = typeof appRouter;
     
     ```

4. **Set Up API Handler for tRPC**:
   
   - `src/pages/api/trpc/[trpc].ts`
     
     ```typescript
     import { createNextApiHandler } from '@trpc/server/adapters/next';
     import { appRouter } from '../../../server/trpc/router';
     import { createContext } from '../../../server/trpc/context';
     export default createNextApiHandler({
     router: appRouter,
     createContext,
     });
     ```

---

### Step 4: Client-Side Setup with tRPC

Configure the tRPC client and integrate it with Next.js and React Query.

1. **Create tRPC Client Utility**:
   
   - `src/utils/trpc.ts`
     
     ```typescript
     import { httpBatchLink } from '@trpc/client';
     import { createTRPCReact } from '@trpc/react-query';
     import type { AppRouter } from '../server/trpc/router';
     export const trpc = createTRPCReact<AppRouter>();
     export const TRPCProvider = trpc.Provider;
     export function getTRPCClient() {
     return trpc.createClient({
     
       links: [
         httpBatchLink({
           url: '/api/trpc',
         }),
       ],
     
     });
     }
     
     ```

2. **Wrap App with Providers**:
   
   - `src/pages/_app.tsx`
     
     ```typescript
     import { AppProps } from 'next/app';
     import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
     import { getTRPCClient, TRPCProvider } from '../utils/trpc';
     import { SessionProvider } from 'next-auth/react';
     const queryClient = new QueryClient();const trpcClient = getTRPCClient();export default function App({ Component, pageProps }: AppProps) { return (
     <SessionProvider session={pageProps.session}>
         <QueryClientProvider client={queryClient}>
           <TRPCProvider client={trpcClient} queryClient={queryClient}>
             <Component {...pageProps} />
           </TRPCProvider>
         </QueryClientProvider>
       </SessionProvider>
       * );}
     ```
   
   **Create Task Management UI**:

3. - `src/pages/index.tsx`
     
     ```typescript
     import { useState } from 'react';
     import { trpc } from '../utils/trpc';
     import { useSession, signOut } from 'next-auth/react';
     import { useRouter } from 'next/router';
     export default function Home() { const { data: session, status } = useSession(); 
     const router = useRouter(); const [title, setTitle] = useState(''); 
     const [description, setDescription] = useState(''); 
     const { data: tasks, isLoading, refetch } = trpc.task.getAll.useQuery(undefined, {
         enabled: !!session,
     }); 
     const createTask = trpc.task.create.useMutation({ onSuccess: () => refetch() }); 
     const updateTask = trpc.task.update.useMutation({ onSuccess: () => refetch() }); 
     const deleteTask = trpc.task.delete.useMutation({ onSuccess: () => refetch() }); 
     if (status === 'loading') return <div>Loading...</div>; if (!session) {
     router.push('/login');
     return null;
     } 
     const handleCreate = async (e: React.FormEvent) => {
     e.preventDefault();
     await createTask.mutateAsync({ title, description });
     setTitle('');
     setDescription('');
     }; 
     const toggleComplete = async (id: string, completed: boolean) => {
     await updateTask.mutateAsync({ id, completed: !completed });
     }; 
     const handleDelete = async (id: string) => {
     await updateTask.mutateAsync({ id, completed: !completed });
     }; 
     const handleDelete = async (id: string) => {
     await deleteTask.mutateAsync({ id });
     }; 
     return (
     <div>
       <h1>Task Manager</h1>
       <p>Welcome, {session.user?.name || session.user?.email}</p>
       <button onClick={() => signOut({ callbackUrl: '/login' })}>Logout</button>
     
       <h2>Create Task</h2>
       <form onSubmit={handleCreate}>
         <input
           type="text"
           value={title}
           onChange={(e) => setTitle(e.target.value)}
           placeholder="Title"
           required
         />
         <input
           type="text"
           value={description}
           onChange={(e) => setDescription(e.target.value)}
           placeholder="Description"
         />
         <button type="submit">Add Task</button>
       </form>
     
       <h2>Your Tasks</h2>
       {isLoading ? (
         <p>Loading tasks...</p>
       ) : (
         <ul>
           {tasks?.map((task) => (
             <li key={task.id}>
               <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
                 {task.title} - {task.description || 'No description'}
               </span>
               <button onClick={() => toggleComplete(task.id, task.completed)}>
                 {task.completed ? 'Mark Incomplete' : 'Mark Complete'}
               </button>
               <button onClick={() => handleDelete(task.id)}>Delete</button>
             </li>
           ))}
         </ul>
       )}
     </div>
     );
     }
     ```

---

### Step 5: Run the Application

1. Start the development server:
   
   ```bash
   npm run dev
   ```

2. Open `http://localhost:3000`. You'll be redirected to `/login`. Register a user, log in, and start managing tasks.

**Note**: For simplicity, passwords are stored in plain text. In production, use a library like `bcrypt` to hash passwords.

---

### Detailed Sequence Diagram for Task Creation

Here’s a text-based sequence diagram for creating a task (`task.create`), illustrating the flow from client to server and back.

```
Participants:
- Client: React Component (src/pages/index.tsx)
- tRPC Client: React Query Hook (trpc.task.create.useMutation)
- tRPC Router: Server API Handler (src/pages/api/trpc/[trpc].ts)
- Middleware: Authentication Check (src/server/trpc/middleware.ts)
- Procedure: Backend Logic (src/server/trpc/router.ts - task.create)
- Prisma: Database ORM (ctx.prisma)
- Database: SQLite (dev.db)

Flow:
1. Client -> tRPC Client: Trigger mutation with trpc.task.create.mutateAsync({ title: "New Task", description: "Details" })
   Note: User submits the form in the React component, calling the tRPC mutation hook.
2. tRPC Client -> tRPC Router: Send HTTP POST to /api/trpc/task.create with payload { title: "New Task", description: "Details" }
   Note: tRPC client serializes the request into an HTTP call via httpBatchLink.
3. tRPC Router -> Middleware: Check if user is authenticated via ctx.userId
   Note: Middleware (isAuthenticated) verifies the session; throws UNAUTHORIZED error if no userId.
4. Middleware -> tRPC Router: Pass control to procedure if authenticated
   Note: Middleware forwards the request with updated context.
5. tRPC Router -> Procedure: Call task.create with input { title: "New Task", description: "Details" } and ctx
   Note: Router invokes the procedure with validated input (via Zod schema).
6. Procedure -> Prisma: Execute ctx.prisma.task.create({ data: { title, description, userId: ctx.userId } })
   Note: Procedure uses Prisma ORM to insert a new task record, linking it to the authenticated user.
7. Prisma -> Database: Insert new task record into tasks table
   Note: Prisma translates the operation into an SQL INSERT query for SQLite.
8. Database -> Prisma: Return newly created task data { id, title, description, userId, ... }
   Note: Database confirms the insertion and returns the record.
9. Prisma -> Procedure: Return task object to procedure
   Note: Prisma passes the created task back to the procedure logic.
10. Procedure -> tRPC Router: Return task data to router
    Note: Procedure sends the result back for response formatting.
11. tRPC Router -> tRPC Client: Send HTTP response with JSON { id, title, description, ... }
    Note: Router serializes the task data into JSON and responds over HTTP.
12. tRPC Client -> Client: Update React state and trigger refetch of tasks
    Note: Mutation hook resolves, triggering the onSuccess callback to refetch task list via trpc.task.getAll.
13. Client -> Client: Render updated UI with new task
    Note: Component re-renders to display the newly created task in the list.
```

---

### Key Real-World Features in This Example

1. **Authentication**:
   
   - Uses NextAuth.js for session management, ensuring only authenticated users can access task endpoints via `protectedProcedure`.

2. **Database Integration**:
   
   - Prisma ORM provides type-safe database access, abstracting SQL operations for scalability to other databases (e.g., PostgreSQL).

3. **Authorization**:
   
   - Ensures users can only access or modify their own tasks by checking `userId` in procedures.

4. **Type Safety**:
   
   - tRPC ensures end-to-end type safety between client and server, with Zod for input validation.

5. **Error Handling**:
   
   - Middleware throws `TRPCError` for unauthorized access, and procedures handle task ownership checks.

6. **Scalability**:
   
   - Router structure (`task.getAll`, `task.create`, etc.) supports easy addition of new endpoints or nested routers.

---

### Production Considerations

- **Security**: Hash passwords with `bcrypt` and use secure session storage (e.g., database-backed sessions with NextAuth).
- **Database**: Switch to PostgreSQL or MySQL for production with environment variables for connection strings.
- **Error Logging**: Add logging (e.g., `winston`) for tRPC errors and database failures.
- **Performance**: Implement pagination for `task.getAll` to handle large task lists.
- **Testing**: Add unit tests for tRPC procedures using Jest and integration tests for the UI.

---

### We can also extend in this example

- **Real-Time Updates**: Add subscriptions with tRPC for real-time task updates using WebSocket links.
- **Task Categories**: Introduce a `Category` model and relate tasks to categories.
- **Advanced Auth**: Integrate OAuth providers (Google, GitHub) with NextAuth.js.
- **Frontend Enhancements**: Use a UI library like Material-UI or Tailwind CSS for better styling.
  
  

#### It's sequence diagram:

Below is a text-based arrow diagram representing the sequence of interactions between components, with each component labeled by its specific file name as provided in the Task Management App example. This diagram focuses on the flow for creating a task after user login, mirroring the detailed sequence from the previous response. Arrows (`->`) indicate the direction of requests or data flow between files.

```
Flow for Task Creation in Task Management App (Arrow Diagram with File Names):

1.  Browser               -> src/pages/_app.tsx
    (User accesses app)      (Providers initialize tRPC and session context)
2.  src/pages/_app.tsx    -> src/pages/login.tsx
    (Redirect if not auth)   (Login page rendered for unauthenticated user)
3.  src/pages/login.tsx   -> src/pages/api/auth/[...nextauth].ts
    (Submit credentials)     (NextAuth handles login request)
4.  src/pages/api/auth/[...nextauth].ts -> prisma/schema.prisma
    (Validate credentials)   (Schema defines User model for database query)
5.  prisma/schema.prisma  -> dev.db
    (Query user data)        (SQLite database file for user record)
6.  dev.db                -> prisma/schema.prisma
    (Return user data)       (Schema relays data via Prisma)
7.  prisma/schema.prisma  -> src/pages/api/auth/[...nextauth].ts
    (Pass user data)         (NextAuth confirms authentication)
8.  src/pages/api/auth/[...nextauth].ts -> Browser
    (Set session cookie)     (Redirect to homepage after login)
9.  Browser               -> src/pages/index.tsx
    (Load homepage)          (Task manager UI for authenticated user)
10. src/pages/index.tsx   -> src/utils/trpc.ts
    (Trigger task creation)  (tRPC client hook for mutation: trpc.task.create)
11. src/utils/trpc.ts     -> src/pages/api/trpc/[trpc].ts
    (Send HTTP request)      (tRPC API handler receives client request)
12. src/pages/api/trpc/[trpc].ts -> src/server/trpc/context.ts
    (Initialize context)     (Context sets up session and Prisma)
13. src/server/trpc/context.ts -> src/pages/api/auth/[...nextauth].ts
    (Fetch session data)     (NextAuth retrieves session for userId)
14. src/pages/api/auth/[...nextauth].ts -> src/server/trpc/context.ts
    (Return session data)    (Context updated with userId)
15. src/server/trpc/context.ts -> src/pages/api/trpc/[trpc].ts
    (Provide context)        (API handler uses context for routing)
16. src/pages/api/trpc/[trpc].ts -> src/server/trpc/router.ts
    (Route to procedure)     (Router matches request to task.create)
17. src/server/trpc/router.ts -> src/server/trpc/middleware.ts
    (Check authentication)   (Middleware ensures user is authenticated)
18. src/server/trpc/middleware.ts -> src/server/trpc/router.ts
    (Proceed if authenticated) (Router continues to procedure)
19. src/server/trpc/router.ts -> src/server/trpc/router.ts
    (Execute task.create)    (Procedure logic for task creation)
20. src/server/trpc/router.ts -> src/server/trpc/context.ts
    (Access Prisma client)   (Context provides Prisma for database ops)
21. src/server/trpc/context.ts -> prisma/schema.prisma
    (Create task record)     (Prisma schema defines Task model)
22. prisma/schema.prisma  -> dev.db
    (Insert task data)       (SQLite database stores new task)
23. dev.db                -> prisma/schema.prisma
    (Return task data)       (Database confirms task creation)
24. prisma/schema.prisma  -> src/server/trpc/context.ts
    (Relay task data)        (Prisma returns data to context)
25. src/server/trpc/context.ts -> src/server/trpc/router.ts
    (Pass task to procedure) (Procedure receives created task)
26. src/server/trpc/router.ts -> src/pages/api/trpc/[trpc].ts
    (Return task data)       (API handler formats response)
27. src/pages/api/trpc/[trpc].ts -> src/utils/trpc.ts
    (Send HTTP response)     (tRPC client receives task data)
28. src/utils/trpc.ts     -> src/pages/index.tsx
    (Update state)           (React component updates with new task)
29. src/pages/index.tsx   -> Browser
    (Render UI)              (Display updated task list to user)
```

This arrow diagram maps the flow of execution across all relevant files in the project, ensuring each component is represented by its specific file name or resource (e.g., `dev.db` for the SQLite database). It captures the complete lifecycle from app initialization, through authentication, to task creation. If you'd like a focus on a different operation (e.g., task deletion) or a different format, let me know!



Let me explain in detail how these two code snippets—`src/utils/trpc.ts` (the tRPC client setup) and `src/pages/api/trpc/[trpc].ts` (the tRPC server API handler)—are connected to each other in the context of a tRPC-based application like the Task Management App from the previous examples. I'll break this down step by step, covering their roles, how they interact, and the flow of data between them, ensuring clarity on their relationship.

### Overview of the Two Files and Their Purpose

1. **Client-Side (`src/utils/trpc.ts`)**:
   
   - This file sets up the tRPC client, which is used by your frontend (React components in Next.js) to make type-safe API calls to the server.
   - It provides the mechanism for the client (browser) to send requests to the server and receive responses, abstracting away the raw HTTP details and making API calls feel like local function calls.

2. **Server-Side (`src/pages/api/trpc/[trpc].ts`)**:
   
   - This file sets up the tRPC API handler, which is the server-side endpoint that listens for incoming requests from the client, processes them via the defined router and context, and sends back responses.
   - It acts as the bridge between the client's HTTP requests and the backend logic defined in your tRPC router.

### Detailed Explanation of Each File's Components

#### 1. Client-Side (`src/utils/trpc.ts`)

```typescript
import { httpBatchLink } from '@trpc/client';
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '../server/trpc/router';

// Initialize tRPC React client
export const trpc = createTRPCReact<AppRouter>();

// Provider configuration for Next.js
export const TRPCProvider = trpc.Provider;

// Client configuration (used in _app.tsx)
export function getTRPCClient() {
  return trpc.createClient({
    links: [
      httpBatchLink({
        url: '/api/trpc', // API endpoint
      }),
    ],
  });
}
```

- **`createTRPCReact<AppRouter>()`**:
  
  - This initializes a tRPC client specifically for React, typed with `AppRouter` (imported from the server-side router definition). This ensures type safety, meaning the client knows exactly what endpoints and data structures are available on the server (e.g., `trpc.task.create` or `trpc.task.getAll`).
  - The result, `trpc`, is a utility that provides React hooks like `useQuery` and `useMutation` for interacting with the server.

- **`TRPCProvider = trpc.Provider`**:
  
  - This is a React component that wraps your entire app (used in `_app.tsx`) to provide the tRPC client context to all components. It ensures that components can access tRPC hooks without additional setup.

- **`getTRPCClient()` and `httpBatchLink`**:
  
  - `getTRPCClient()` creates a tRPC client instance with a configuration for how it communicates with the server.
  - `httpBatchLink({ url: '/api/trpc' })` is the transport mechanism. It tells the client to send all requests as HTTP requests to the `/api/trpc` endpoint on the server. The "batch" part means multiple tRPC calls can be combined into a single HTTP request for efficiency.
  - This `url: '/api/trpc'` is the critical connection point—it specifies the exact server endpoint the client will communicate with, which is defined in the server-side file.

#### 2. Server-Side (`src/pages/api/trpc/[trpc].ts`)

```typescript
import { createNextApiHandler } from '@trpc/server/adapters/next';
import { appRouter } from '../../../server/trpc/router';
import { createContext } from '../../../server/trpc/context';

// Export API handler for tRPC
export default createNextApiHandler({
  router: appRouter,
  createContext,
});
```

- **`createNextApiHandler`**:
  
  - This function from `@trpc/server/adapters/next` creates an API handler tailored for Next.js. It turns your tRPC router into a Next.js API route that can handle incoming HTTP requests.
  - In Next.js, API routes are defined under `pages/api/`, and the `[trpc].ts` naming ensures dynamic routing, meaning any request to `/api/trpc/*` (e.g., `/api/trpc/task.create`) is handled by this file.

- **`router: appRouter`**:
  
  - This connects the API handler to your tRPC router (`appRouter` from `../../../server/trpc/router`), which defines all the available endpoints (procedures like `task.create` or `task.getAll`). The handler uses this router to determine how to process incoming requests.

- **`createContext`**:
  
  - This connects the API handler to a context function (`createContext` from `../../../server/trpc/context`), which provides shared data (like database access via Prisma or user session info) to all procedures when they execute. The handler uses this to initialize the context for each request.

### How They Are Connected: The Client-Server Relationship

The connection between these two files is primarily through the **HTTP endpoint `/api/trpc`**, which serves as the communication bridge between the client and server. Let's walk through how they interact:

1. **Client Initiates Communication**:
   
   - In `src/utils/trpc.ts`, the `httpBatchLink` is configured with `url: '/api/trpc'`. This tells the tRPC client to send all its requests to this specific endpoint on the server.
   - For example, when a React component calls `trpc.task.create.useMutation()` (in `src/pages/index.tsx`), the tRPC client serializes the request (e.g., the procedure name `task.create` and input data like `{ title: "New Task" }`) into an HTTP POST request directed at `/api/trpc/task.create`.

2. **Server Receives and Processes the Request**:
   
   - On the server side, `src/pages/api/trpc/[trpc].ts` is set up as a Next.js API route that listens for requests to `/api/trpc/*`. The dynamic route parameter `[trpc]` captures the specific endpoint (like `task.create`).
   - The `createNextApiHandler` uses the provided `appRouter` to map the incoming request path (e.g., `task.create`) to the correct procedure in the router (defined in `../../../server/trpc/router.ts`).
   - It also uses `createContext` to build a context object (e.g., with user session and database access) for the procedure to use during execution.

3. **Data Flow and Response**:
   
   - Once the server processes the request (e.g., creates a task in the database via the `task.create` procedure), the result (or error) is serialized into a JSON response by `createNextApiHandler`.
   - This response is sent back over HTTP to the client at the same `/api/trpc` endpoint.
   - The tRPC client in `src/utils/trpc.ts` receives this HTTP response, deserializes it, and updates the React component's state via the hook (e.g., resolving the mutation with the new task data).

### Detailed Flow of Connection (Step-by-Step Interaction)

Here’s a focused sequence showing how a single request (e.g., creating a task) connects these two files:

```
Participants (File Names):
- Client_Component: src/pages/index.tsx (React UI)
- Client_tRPC: src/utils/trpc.ts (tRPC Client)
- Server_API_Handler: src/pages/api/trpc/[trpc].ts (API Endpoint)
- Server_Router: src/server/trpc/router.ts (Procedure Logic)
- Server_Context: src/server/trpc/context.ts (Context Setup)

Flow:
1. Client_Component (src/pages/index.tsx) -> Client_tRPC (src/utils/trpc.ts):
   Action: User submits a form, triggering trpc.task.create.useMutation({ title: "New Task" }).
   Note: React component uses the tRPC hook provided by trpc.createClient.

2. Client_tRPC (src/utils/trpc.ts) -> Server_API_Handler (src/pages/api/trpc/[trpc].ts):
   Action: tRPC client sends an HTTP POST request to /api/trpc/task.create with serialized payload.
   Note: httpBatchLink targets the URL /api/trpc as defined in getTRPCClient().

3. Server_API_Handler (src/pages/api/trpc/[trpc].ts) -> Server_Context (src/server/trpc/context.ts):
   Action: API handler initializes context using createContext for the request.
   Note: createNextApiHandler calls createContext to get session and database access.

4. Server_API_Handler (src/pages/api/trpc/[trpc].ts) -> Server_Router (src/server/trpc/router.ts):
   Action: Handler routes the request to the task.create procedure in appRouter.
   Note: createNextApiHandler uses router: appRouter to map task.create to the correct logic.

5. Server_Router (src/server/trpc/router.ts) -> Server_API_Handler (src/pages/api/trpc/[trpc].ts):
   Action: Procedure executes, returns task data (or error) to the handler.
   Note: Router sends the result back for HTTP response formatting.

6. Server_API_Handler (src/pages/api/trpc/[trpc].ts) -> Client_tRPC (src/utils/trpc.ts):
   Action: Handler sends HTTP response with JSON data back to the client.
   Note: Response is sent to the client over the same /api/trpc connection.

7. Client_tRPC (src/utils/trpc.ts) -> Client_Component (src/pages/index.tsx):
   Action: tRPC client resolves the mutation, updating React state with new task data.
   Note: Hook updates UI via React Query integration.
```

### Key Connection Points

- **URL Endpoint (`/api/trpc`)**:
  
  - The client (`src/utils/trpc.ts`) explicitly points to `/api/trpc` via `httpBatchLink`. This matches the server-side API route defined by the file location and naming of `src/pages/api/trpc/[trpc].ts` in Next.js. This shared endpoint is how they "find" each other.

- **Type Safety via `AppRouter`**:
  
  - The client uses `createTRPCReact<AppRouter>()` with the server's `AppRouter` type (imported from `../server/trpc/router`). This ensures the client knows exactly which endpoints exist on the server, creating a tight, type-safe connection without manual API contracts.

- **Request-Response Cycle**:
  
  - The client sends requests using HTTP (via `httpBatchLink`), and the server processes them with `createNextApiHandler`, completing the communication loop. Both sides handle serialization/deserialization automatically, so developers work with typed data instead of raw HTTP.

### Analogy for Better Understanding

Think of this connection as a **phone call system**:

- The client (`src/utils/trpc.ts`) is like a person with a phone, dialing a specific number (`/api/trpc`) to reach someone.
- The server (`src/pages/api/trpc/[trpc].ts`) is like the switchboard operator at that number, receiving the call, figuring out who to connect to (via `appRouter`), and setting up the environment for the conversation (via `createContext`).
- The conversation happens over the same line (`/api/trpc`), with the client sending requests (questions) and the server sending responses (answers).

### Why This Connection Matters

- **End-to-End Type Safety**: The shared `AppRouter` type ensures that what the client expects matches what the server provides, reducing runtime errors.
- **Simplicity**: Developers don’t need to write REST endpoints or handle raw HTTP requests manually; tRPC abstracts this into function-like calls.
- **Performance**: `httpBatchLink` optimizes requests by batching multiple calls, and the server handler efficiently routes them, minimizing overhead.

### Summary

- `src/utils/trpc.ts` (client) and `src/pages/api/trpc/[trpc].ts` (server) are connected via the HTTP endpoint `/api/trpc`, which serves as the communication channel.
- The client sends requests to this endpoint using `httpBatchLink`, and the server listens for and processes these requests using `createNextApiHandler`.
- They rely on shared type definitions (`AppRouter`) for consistency and use React Query for state management on the client side, while the server leverages the router and context for logic and environment setup.

If you have further questions, want to dive deeper into a specific part (e.g., batching with `httpBatchLink` or context setup), or need a visual representation suggestion, just let me know!





Browser -> src/pages/_app.tsx -> src/pages/login.tsx -> src/pages/api/auth/[...nextauth].ts -> prisma/schema.prisma -> dev.db -> prisma/schema.prisma -> src/pages/api/auth/[...nextauth].ts -> Browser -> src/pages/index.tsx -> src/utils/trpc.ts -> src/pages/api/trpc/[trpc].ts -> src/server/trpc/context.ts -> src/pages/api/auth/[...nextauth].ts  -> src/server/trpc/context.ts -> src/pages/api/trpc/[trpc].ts -> src/server/trpc/router.ts -> src/server/trpc/middleware.ts  -> src/server/trpc/router.ts -> src/server/trpc/router.ts -> src/server/trpc/context.ts -> prisma/schema.prisma -> dev.db -> prisma/schema.prisma -> src/server/trpc/context.ts -> src/server/trpc/router.ts -> src/pages/api/trpc/[trpc].ts -> src/utils/trpc.ts -> src/pages/index.tsx -> Browser



It's diagram of TRPC having middleware and context processing steps combined into a single processing step before routing

![833deb43-87f4-4da3-b6bb-dbe7efad9c81](file:///C:/Users/HP/OneDrive/Pictures/Typedown/833deb43-87f4-4da3-b6bb-dbe7efad9c81.png)

The level of detail I want to show about the server-side internal calls and middleware processing are: 

1. Middleware authorization checks and branching

2. Context initialization with session and database 

3. clientInternal server-side procedure calls (e.g., logging)

4. Router merging and procedure dispatching

![b9755ee2-4978-42f2-8bfc-055747f1413d](file:///C:/Users/HP/OneDrive/Pictures/Typedown/b9755ee2-4978-42f2-8bfc-055747f1413d.png)

I'll provide a detailed explanation of each topic related to tRPC, as outlined in your list, under the categories of **Getting Started**, **Backend Usage**, **Server Side Calls**, **Client Usage**, and **Links**. For each topic, I'll include a concise description, its importance, and a small, focused example based on a simplified Task Management App (similar to the previous examples). At the end, I'll combine them into a cohesive analogy and provide a comprehensive sequence diagram showing how they interact in a full application flow.

Since this is a broad and extensive request, I'll keep each section brief but actionable, ensuring clarity and relevance. If you need deeper dives into any specific topic, let me know!

---

### Getting Started

#### 1. Getting Started

- **Description**: An introduction to tRPC, a TypeScript-first library for building type-safe APIs between client and server without manual schema definitions.

- **Importance**: Helps new users understand tRPC's purpose—reducing boilerplate and ensuring type safety.

- **Example**: Install tRPC in a Next.js app.
  
  ```bash
  npm install @trpc/server @trpc/client @trpc/next @trpc/react-query zod
  ```
  
  This sets up the core dependencies for a tRPC project.

#### 2. Concepts

- **Description**: Core ideas behind tRPC, like type-safe RPC (Remote Procedure Call), shared types, and no need for external API schemas (e.g., OpenAPI).

- **Importance**: Establishes why tRPC differs from REST or GraphQL by making API calls feel like local function calls.

- **Example**: tRPC uses TypeScript inference to share types between client and server.
  
  ```typescript
  // Server: src/server/trpc/router.ts
  export const appRouter = router({
    greet: publicProcedure.query(() => "Hello, World!"),
  });
  export type AppRouter = typeof appRouter;
  // Client: Auto-inherits types from AppRouter
  ```

#### 3. Quickstart

- **Description**: A fast guide to setting up a basic tRPC project with minimal configuration.

- **Importance**: Allows beginners to see tRPC in action quickly.

- **Example**: Basic setup in Next.js (client and server).
  
  ```typescript
  // Server: src/pages/api/trpc/[trpc].ts
  import { createNextApiHandler } from '@trpc/server/adapters/next';
  import { appRouter } from '../../../server/trpc/router';
  export default createNextApiHandler({ router: appRouter });
  // Client: src/utils/trpc.ts
  import { createTRPCReact } from '@trpc/react-query';
  export const trpc = createTRPCReact<AppRouter>();
  ```

#### 4. Videos & Community Resources

- **Description**: Links to tutorials, talks, and forums for learning tRPC (e.g., official docs, GitHub, Discord).
- **Importance**: Provides external learning materials and community support.
- **Example**: Check the tRPC GitHub for updates.
  - URL: `https://github.com/trpc/trpc`

#### 5. Example Apps

- **Description**: Pre-built tRPC apps showcasing real-world usage (e.g., Todo apps, chat apps).

- **Importance**: Offers templates for learning and starting projects.

- **Example**: Clone the tRPC Next.js example.
  
  ```bash
  git clone https://github.com/trpc/examples-next-prisma-todo.git
  ```

---

### Backend Usage

#### 6. Define Routers

- **Description**: Routers are the central structure in tRPC to organize API endpoints (procedures) on the server.

- **Importance**: Groups related functionality into a single router for clarity.

- **Example**: Define a router for tasks.
  
  ```typescript
  // src/server/trpc/router.ts
  import { router } from './trpc';
  export const appRouter = router({
    task: {
      // Procedures go here
    }
  });
  ```

#### 7. Define Procedures

- **Description**: Procedures are the individual API endpoints (queries for reads, mutations for writes) within a router.

- **Importance**: Represents callable server functions.

- **Example**: A query to get tasks.
  
  ```typescript
  // src/server/trpc/router.ts
  import { publicProcedure } from './trpc';
  export const appRouter = router({
    task: {
      getAll: publicProcedure.query(() => [{ id: 1, title: "Task 1" }]),
    }
  });
  ```

#### 8. Input & Output Validators

- **Description**: Validate incoming data and outgoing responses using libraries like Zod for type safety.

- **Importance**: Ensures data integrity and prevents runtime errors.

- **Example**: Validate input for creating a task.
  
  ```typescript
  import { z } from 'zod';
  export const appRouter = router({
    task: {
      create: publicProcedure.input(z.object({ title: z.string() })).mutation(({ input }) => ({
        id: 2,
        title: input.title,
      })),
    }
  });
  ```

#### 9. Non-JSON Inputs (FormData, File, Blob)

- **Description**: Handling non-JSON inputs like file uploads in tRPC using custom parsing.

- **Importance**: Extends tRPC beyond simple JSON data.

- **Example**: Handle file upload (simplified).
  
  ```typescript
  export const appRouter = router({
    task: {
      uploadImage: publicProcedure.input(z.any()).mutation(({ input }) => "File received"),
    }
  });
  ```

#### 10. Merging Routers

- **Description**: Combine multiple routers into a single router for modularity.

- **Importance**: Keeps code organized as the app grows.

- **Example**: Merge task and user routers.
  
  ```typescript
  const taskRouter = router({ task: { getAll: publicProcedure.query(() => []) } });
  const userRouter = router({ user: { get: publicProcedure.query(() => ({ id: 1 })) } });
  export const appRouter = router({ ...taskRouter, ...userRouter });
  ```

#### 11. Context

- **Description**: A shared object passed to procedures, often containing database clients or user info.

- **Importance**: Provides a way to access shared resources per request.

- **Example**: Add user session to context.
  
  ```typescript
  // src/server/trpc/context.ts
  export async function createContext() {
    return { userId: "user123" };
  }
  ```

#### 12. Middlewares

- **Description**: Functions that run before procedures to handle cross-cutting concerns like authentication.

- **Importance**: Enforces rules like access control.

- **Example**: Middleware to check authentication.
  
  ```typescript
  const isAuthenticated = t.middleware(({ next, ctx }) => {
    if (!ctx.userId) throw new TRPCError({ code: 'UNAUTHORIZED' });
    return next();
  });
  ```

#### 13. Hosting tRPC with Adapters

- **Description**: Use adapters to host tRPC on different environments (e.g., Next.js, Express).

- **Importance**: Makes tRPC flexible across platforms.

- **Example**: Use Next.js adapter.
  
  ```typescript
  // src/pages/api/trpc/[trpc].ts
  import { createNextApiHandler } from '@trpc/server/adapters/next';
  export default createNextApiHandler({ router: appRouter });
  ```

---

### Server Side Calls

#### 14. Server Side Calls

- **Description**: Call tRPC procedures directly on the server (e.g., during SSR or API routes).

- **Importance**: Enables server-to-server communication without HTTP.

- **Example**: Call a procedure on the server.
  
  ```typescript
  // src/pages/api/some-route.ts
  const result = await appRouter.createCaller({ userId: "user123" }).task.getAll();
  ```

#### 15. Authorization

- **Description**: Implement access control to restrict procedure access based on user permissions.

- **Importance**: Secures sensitive operations.

- **Example**: Restrict task creation.
  
  ```typescript
  const createTask = protectedProcedure.mutation(({ ctx }) => {
    if (!ctx.userId) throw new Error("Unauthorized");
    return { id: 1, title: "New Task" };
  });
  ```

#### 16. Error Handling

- **Description**: Manage errors in procedures using `TRPCError` for standardized responses.

- **Importance**: Provides consistent error feedback to clients.

- **Example**: Throw an error if task not found.
  
  ```typescript
  import { TRPCError } from '@trpc/server';
  const getTask = publicProcedure.query(() => {
    throw new TRPCError({ code: 'NOT_FOUND', message: 'Task not found' });
  });
  ```

#### 17. Error Formatting

- **Description**: Customize how errors are formatted in responses.

- **Importance**: Tailors error messages for client consumption.

- **Example**: Custom error message.
  
  ```typescript
  const errorFormatter = ({ error }) => ({ message: `Error: ${error.message}` });
  ```

#### 18. Data Transformers

- **Description**: Transform data before sending (e.g., serialize Date objects).

- **Importance**: Ensures data compatibility across client-server.

- **Example**: Use superjson for complex data.
  
  ```typescript
  import superjson from 'superjson';
  const client = trpc.createClient({ transformer: superjson });
  ```

#### 19. Metadata

- **Description**: Add metadata to procedures (e.g., for documentation or logging).

- **Importance**: Useful for debugging or tooling.

- **Example**: Add metadata to a procedure.
  
  ```typescript
  const getTask = publicProcedure.meta({ description: "Get all tasks" }).query(() => []);
  ```

#### 20. Response Caching

- **Description**: Cache responses to improve performance for frequent queries.

- **Importance**: Reduces server load.

- **Example**: Cache a query response.
  
  ```typescript
  const getTasks = publicProcedure.query(() => [], {
    staleTime: 60 * 1000, // 1 minute
  });
  ```

#### 21. Subscriptions

- **Description**: Enable real-time updates using WebSocket subscriptions.

- **Importance**: Supports live data updates.

- **Example**: Subscribe to task updates.
  
  ```typescript
  const taskUpdates = publicProcedure.subscription(() => observable(() => {}));
  ```

#### 22. WebSockets

- **Description**: Configure WebSocket links for subscriptions.

- **Importance**: Powers real-time features.

- **Example**: Set up WebSocket link.
  
  ```typescript
  import { wsLink } from '@trpc/client';
  const client = trpc.createClient({ links: [wsLink({ url: 'ws://localhost:3001' })] });
  ```

---

### Client Usage

#### 23. TanStack React Query (⭐️)

- **Description**: Integration of tRPC with React Query for state management, caching, and retries.

- **Importance**: Enhances client-side data fetching.

- **Example**: Use React Query with tRPC.
  
  ```typescript
  // src/utils/trpc.ts
  import { createTRPCReact } from '@trpc/react-query';
  export const trpc = createTRPCReact<AppRouter>();
  ```

#### 24. React Query Integration (Classic)

- **Description**: Older method of integrating tRPC with React Query (pre-v10).

- **Importance**: Useful for legacy projects.

- **Example**: Classic integration (simplified).
  
  ```typescript
  import { createTRPCReact } from '@trpc/react';
  export const trpc = createTRPCReact<AppRouter>();
  ```

#### 25. Next.js Integration

- **Description**: tRPC setup specifically for Next.js, including API routes and SSR.

- **Importance**: Optimizes tRPC for Next.js apps.

- **Example**: Provider setup in Next.js.
  
  ```typescript
  // src/pages/_app.tsx
  import { TRPCProvider } from '../utils/trpc';
  <TRPCProvider client={trpcClient}><Component {...pageProps} /></TRPCProvider>
  ```

#### 26. Vanilla Client

- **Description**: Use tRPC without React, for non-React environments.

- **Importance**: Extends tRPC to other frameworks.

- **Example**: Vanilla client setup.
  
  ```typescript
  import { createTRPCClient } from '@trpc/client';
  const client = createTRPCClient<AppRouter>({ links: [httpBatchLink({ url: '/api/trpc' })] });
  ```

---

### Links

#### 27. Create Custom Header

- **Description**: Add custom headers to tRPC requests (e.g., for auth tokens).

- **Importance**: Supports custom authentication.

- **Example**: Add a custom header.
  
  ```typescript
  const client = trpc.createClient({
    links: [httpBatchLink({ url: '/api/trpc', headers: { 'x-custom': 'value' } })]
  });
  ```

#### 28. CORS & Cookies

- **Description**: Handle Cross-Origin Resource Sharing and cookies for tRPC requests.

- **Importance**: Enables cross-domain requests and session management.

- **Example**: Configure CORS in Next.js.
  
  ```typescript
  // src/pages/api/trpc/[trpc].ts
  export default createNextApiHandler({
    router: appRouter,
    onError: ({ req, res }) => res.setHeader('Access-Control-Allow-Origin', '*')
  });
  ```

---

### Analogy: tRPC as a City Infrastructure

Think of tRPC as a **city infrastructure system** for communication between people (client) and services (server):

- **Getting Started (Concepts, Quickstart, etc.)**: Learning the city's layout and rules—how to navigate and use services.
- **Backend Usage (Routers, Procedures, Context, etc.)**: The city's administrative offices—defining departments (routers), specific desks (procedures), and shared resources (context) for handling requests.
- **Server Side Calls (Authorization, Error Handling, etc.)**: Internal city operations—security checks (authorization), complaint handling (errors), and real-time alerts (subscriptions).
- **Client Usage (React Query, Next.js, etc.)**: Residents' tools—phones or apps (React Query hooks) to request services, tailored for specific neighborhoods (Next.js).
- **Links (Custom Headers, CORS)**: Special delivery instructions—ensuring requests can cross city borders (CORS) or include ID cards (headers).

In this analogy, a resident (client) uses a phone app (React Query) to request a new permit (task creation) by dialing a central office number (`/api/trpc`). The office (server router) routes the call to the right desk (procedure), checks ID (middleware), uses city records (context), handles issues (error handling), and sends a response or live updates (subscriptions) back through the same line.



# 


