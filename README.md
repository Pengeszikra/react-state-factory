# react-state-factory
react-state-factory is a minimalist library that helps organize mid-complex state handling with type-guarded dispatch-capable actions. It leverages `useReducer` and TypeScript to bring type safety and structure to your application's state management.

## Installation

```sh
npm add react-state-factory
# or yarn
yarn add react-state-factory
# or pnpm
pnpm add react-state-factory
```

need to be sure the following package is need to installed: 
- `useStateFactory` : react
- `useSagaFactory`  : react, redux-saga, use-saga-reducer

## Usage

### Declare the set of actions with their types

```ts
// actions.ts

export type ActionMap =
  | { type: "START_APPLICATION", payload: {start: number, id: string } }
  | { type: "PLACE_CONENT", payload: number[] }
  | { type: "ADD_ITEM", payload: number }

// after write export const actions:Labels<ActionMap> = {} can autogenerated this const
// with VS-Code QuickFix option, or any times when ActionMap extend with new entry
export const actions:Labels<ActionMap> = {
  START_APPLICATION : "START_APPLICATION",
  PLACE_CONTENT : "PLACE_CONTENT",
  ADD_ITEM : "ADD_ITEM",
};
```

```ts
// reducer.ts

export type SimpleState = {
  content: number[],
  start: number,
  id: string,
}

export const gameReducer:Reducer<SimpleState, ActionMap> = (state  { type, payload }) => {
  switch (type) {
    case actions.LETS_PLAY: return {...state, isReady: payload};
    case actions.START_APPLICATION: return {...state, id: payload.id, start: payload: start};
    case actions.PLACE_CONENT: return {...state, content: payload };
    case actions.ADD_ITEM: return {...state, content: [...state.content, payload]};
    default return state;
  }
```

### SimpleComponent example 

```tsx
// SimpleComponent.tsx

import { FC, useEffect } from 'react';
import { useStateFactory } from 'react-state-factory';
import { actions, ActionMap } from './actions';

export const SimpleComponent: FC = () => {

  const [state, put] = useStateFactory(reducer, initialState, actions);

  useEffect(() => {
    put.START_APPLICATION({
      start: Date.now(),
      id: "-basic-app-uui-",
    });
  }, [put]);

  return (
    <main className="bg-black text-green-400 min-h-screen grid place-items-center relative">
      <pre>{JSON.stringify(state, null, 2)}</pre>
    </main>
  );
}
```

In this example, `useStateFactory` is used to create a `state` and a `put` function. The `state` is the current state of the application, and `put` is a function that dispatches actions to the reducer. The `put` function is created using the `actions` enum and is type-safe, meaning you will get TypeScript errors if you try to dispatch an action with the wrong payload type.

### Example of use in saga

```ts
// exampleGenerator.ts

import { typedPutActionMapFactory } from 'react-state-factory';
import { actions, ActionMap } from './actions';

const run = typedPutActionMapFactory<typeof actions, ActionMap>(actions);

export function * exampleGenerator() {
  yield run.START_APPLICATION({
    start: Date.now(), 
    id: Math.random().toString(32).slice(-8)
  });
  yield run.PLACE_CONENT([87, 45, 23, 12]);
  yield run.ADD_ITEM(42);
}
```

In this example, `typedPutActionMapFactory` is used to create a `run` function, which is a saga generator that dispatches actions. The `run` function is created using the `actions` enum and is type-safe.

## API Reference

### `useStateFactory`

```ts
export const useStateFactory = <
  AM extends ActionType<any>, 
  ST,
  PT extends Labels<AM>,
>(
  reducer: (st: ST, action: AM) => ST,
  initialState: ST,
  labels: PT,
)  => {
  type SagaResult = [state: ST, dispatch: Dispatch<AM>];
  const [state, dispatch]: SagaResult = useReducer(reducer, initialState);
  const put = useMemo(
    () => typedActionMapFactory(labels, dispatch), [dispatch, labels]
  );
  return [state, put] as UseFactoryReturn<ST, TypedActionMap<AM, PT>>;
};
```

Creates a `state` and a `put` function.

- `reducer`: A function that takes the current `state` and an `action` and returns the new `state`.
- `initialState`: The initial state of your application.
- `labels`: An object where the keys are the action types and the values are the action type strings.

Returns an array with the `state` and the `put` function.

### `useSagaFactory`

```ts
export const useSagaFactory = <
  AM extends ActionType<any>, 
  ST,
  PT extends Labels<AM>,
>(
  reducer: (st: ST, action: AM) => ST,
  initialState: ST,
  labels: PT,
  saga: Saga,
)  => {
  type SagaResult = [state: ST, dispatch: Dispatch<AM>];
  const [state, dispatch]: SagaResult = useSagaReducer(saga, reducer, initialState);
  const put = useMemo(
    () => typedActionMapFactory(labels, dispatch), [dispatch, labels]
  );
  return [state, put] as UseFactoryReturn<ST, TypedActionMap<AM, PT>>;
};
```

Creates a `state` and a `put` function.

- `reducer`: A function that takes the current `state` and an `action` and returns the new `state`.
- `initialState`: The initial state of your application.
- `labels`: An object where the keys are the action types and the values are the action type strings.
- `saga`: A redux-saga generator function.

Returns an array with the `state` and the `put` function exactly same as in `useStateFactory`.

### `typedPutActionMapFactory`

```ts
export function typedPutActionMapFactory<
  AM extends ActionType<any>
>(
  labels: Labels<AM>
): TypedGeneratorMap<AM, Labels<AM>> {
  return Object.keys(labels).reduce((acc, type) => ({
    ...acc,
    [type]: function * putGenerator(payload: any) {
      yield put({ type, payload });
    }
  }), {} as TypedGeneratorMap<AM, Labels<AM>>);
}
```

Creates `put` function for saga use.

- `labels`: An object where the keys are the action types and the values are the action type strings.

## Discuss

[Simplify your React state management with react-state-factory on dev.to](https://dev.to/pengeszikra/simplify-your-react-state-management-with-react-state-factory-4a14)

[0.0.8 -> 0.0.9 development note](https://dev.to/pengeszikra/typescript-challenge-1ag7)

## Contribution

If you want to contribute to this project, please fork the repository, create a new branch for your work, and open a pull request.

## License

MIT

### npm local test
This is help a lot under npm module development

https://dev.to/scooperdev/use-npm-pack-to-test-your-packages-locally-486e

```sh
npm pack --pack-destination ~
```