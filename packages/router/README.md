# @huse/router

Enpower `react-router-dom` with hooks to interactive with location, state, search params and more.

**NOTE: This packages works only with `v5.x` and `react-router-dom` currently, `react-router-native` and `6.x` version are not supported.**

## useNavigate

A implement of `react-router@6.x`'s `useNavigate` above version `5.x`.

```typescript
type Navigate = <S>(to: Location<S> | string, options?: NavigateOptions<S>) => void;
function useNavigate(): Navigate;
```

## useLocationState

Wrap `location.state` to a react state tuple.

```typescript
type UpdateLocationState<T> = (patch: Partial<T>) => void;
function useLocationState<T>(defaultValue: T): [T, UpdateLocationState<T>];
```

Commonly we store invisible but persist state like list selections inside it.

```jsx
import {useLocationState} from '@huse/router';
import {Checkbox} from 'antd';

const App = () => {
    const [{selected}, setSelection] = useLocationState({selected: []});
    const pushSelected = useCallback(
        id => setSelection({selected: [...selected, id]}),
        [selected, setSelection]
    );
    const removeSelected = useCallback(
        id => {
            const index = selected.indexOf(id);
            setSelection({selected: [...selected.slice(0, index), ...selected.slice(index + 1)]});
        },
        [selected, setSelection]
    );
    const items = Array.from({length: 10}, (v, i) => ({id: i, name: `Item ${i}`}));
    const renderItem = ({id, name}) => (
        <tr key={id}>
            <th>
                <Checkbox onChange={e => (e.target.checked ? pushSelected(id) : removeSelected(id))} />
            </th>
            <td>
                {name}
            </td>
        </tr>
    );

    return (
        <table>
            <tbody>
                {items.map(renderItem)}
            </tbody>
        </table>
    );
};
```

## useSearchParams

Parse location search to a `URLSearchParams` object.

```typescript
function useSearchParams(): URLSearchParams;
```

## useSearchParam

Like `useSearchParams` but get a single param.

```typescript
function useSearchParam(key: string): string | null
```

## useSearchParamAll

Like `useSearchParams` but get a single param as array.

```typescript
function useSearchParamAll(key: string): string[]
```

## useUpdateSearchParams

Get a function to update search params via any object.

```typescript
interface SearchQuery {
    [key: string]: string | string[];
}

interface UpdateSearchParamsOptions {
    replace?: boolean;
    stringify?(query: SearchQuery): string;
}

type UpdateSearchParams = <S>(patch: S) => void;

function useUpdateSearchParams(options?: UpdateSearchParamsOptions): UpdateSearchParams;
```

By default `URLSearchParams#toString` is used to stringify search params to search string, you can use your own `stringify` option using 3rd-party packages like `query-string`:

```javascript
import queryString from 'query-string';

const useUpdateQueryString = () => {
    // Pass a custom stringify function.
    const updateSearchParams = useUpdateSearchParams({stringify: queryString.stringify});
    return updateSearchParams;
};
```

## useSearchParamState

Wrap a single search params as a react state.

```typescript
function useSearchParamState(key: string, options: UpdateSearchParamsOptions = {replace: false}): [string | null, (value: string) => void];
```

When a state is stored in search params, using this hooks works just like `useState`.

```jsx
import {useSearchParamState} from '@huse/router';
import {Button} from 'antd';

const App = () => {
    const [page, setPage] = useSearchParamState('page');
    const pageIndex = parseInt(page ?? '1', 10);
    const nextPage = useCallback(
        () => setPage((pageIndex + 1).toString()),
        [pageIndex, setPage]
    );
    const start = pageIndex * 10 + 1;
    const items = Array.from({length: 10}, (v, i) => ({id: start + i, name: `Item ${start + i}`}));

    return (
        <>
            <ul>
                {items.map(i => <li key={i.id}>{i.name}</li>)}
            </ul>
            <Button onClick={nextPage}>More Items</Button>
        </>
    );
};
