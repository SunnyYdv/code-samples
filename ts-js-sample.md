```tsx
/***
 * Converts array of objects to options
 *
 * arrayToOptions<ArrayItemType>([{ id: 1, title: 'Test' }], 'id', 'title')
 *
 * Result: [{ value: 1, label: 'Test' }]
 */
export const arrayToOptions = <T, V = List.Option['value']>(
  arr: T[],
  valueKey: keyof T | ((item: T) => V),
  labelKey: keyof T | ((item: T) => string)
): List.Option<V>[] => {
  return arr.map(item => {
    return {
      value: typeof valueKey === 'function' ? valueKey(item) : get(item, valueKey),
      label: typeof labelKey === 'function' ? labelKey(item) : get(item, labelKey)
    }
  })
}

/***
 Converts document model fields array to object

 Example:
 const Fields = [{ Key: 'Name', ..., Value: 'John Doe' }],

 getModelValues(Fields)

 Result: { ..., Name: { Value: 'John Doe' }, ... }
 */
export const getModelValues = <T extends Core.Object>(model: DM.Type): T => {
  const { Fields } = model

  return Fields?.reduce((acc, field) => {
    const { DataType, Text, Key } = field

    const isObjectList = DataType === 'ObjectList'

    if (isObjectList) {
      const { Value } = field
      const isArrayValue = Value && Array.isArray(Value) && Value.length

      return {
        ...acc,
        [Key]: isArrayValue ? Value.map(getDocumentModelText) : []
      }
    }

    return {
      ...acc,
      [Key]: Text
    }
  }, {} as T)
}
```

TypeScript

Types for custom api structure for react-query(tanstack-query) and axios

The structure is a tree of nested objects, each object is nested based on the url

```tsx
export namespace Api {
  export type Schema = ApiSchema
  export type Resolver<TResponse = any, TPayload = any> = (payload: TPayload) => Promise<TResponse>

  export type ResponseType<TResolver extends Resolver> = Awaited<ReturnType<TResolver>>
  export type ErrorType = AxiosError<{ Message?: string }>
  export type PayloadType<T extends (...args: any) => any> = T extends (payload: infer P) => any ? P : never

  export type QueryResult<TResolver extends Resolver> = UseQueryResult<ResponseType<TResolver>, ErrorType>
  export type QueryOptions<TResolver extends Resolver> = UseQueryOptions<
    ResponseType<TResolver>,
    ErrorType,
    ResponseType<TResolver>,
    [string, PayloadType<TResolver>]
  >

  export type MutationResult<TResolver extends Resolver> = UseMutationResult<
    ResponseType<TResolver>,
    ErrorType,
    PayloadType<TResolver>
  >
  export type MutationOptions<TResolver extends Resolver> = UseMutationOptions<
    ResponseType<TResolver>,
    ErrorType,
    PayloadType<TResolver>
  >

  export type RequestResolverOptions = {
    alert: AlertContext
    client: QueryClient
  }

  export type RequestResolver<TResolver extends Resolver, TQueryKey extends string = string> = {
    key: TQueryKey
    resolver: Resolver<ResponseType<TResolver>, PayloadType<TResolver> & { signal?: AbortSignal }>
    successMessage?: string
    errorMessage?: string
    onSuccess?: (
      response: ResponseType<TResolver>,
      payload: PayloadType<TResolver>,
      options: RequestResolverOptions
    ) => void
    onError?: (error: ErrorType, payload: PayloadType<TResolver>, options: RequestResolverOptions) => void
  }

  export type RequestData<TQueryKey extends string = string> = {
    Key: TQueryKey
    Response: any
    Payload?: any
  }

  export type RequestsDataTypes = 'Get' | 'Post' | 'Put' | 'Patch' | 'Remove'
  export type RequestsData = {
    [P in RequestsDataTypes]?: RequestData
  }

  export type RequestTypes = 'get' | 'post' | 'put' | 'patch' | 'remove'
  export type RequestTypesDataTypes = {
    get: 'Get'
    post: 'Post'
    put: 'Put'
    patch: 'Patch'
    remove: 'Remove'
  }

  export type Request<
    TRequestsData extends RequestsData,
    TRequestsTypeData extends keyof TRequestsData
  > = TRequestsData[TRequestsTypeData] extends infer RRequestsData
    ? RRequestsData extends RequestData
      ? RequestResolver<Resolver<RRequestsData['Response'], RRequestsData['Payload']>, RRequestsData['Key']>
      : never
    : never

  export type Requests<TRequestsData extends RequestsData> = {
    [P in RequestTypes]?: Request<TRequestsData, RequestTypesDataTypes[P]>
  }

  export type RequiredRequests<
    TRequestsData extends RequestsData,
    TRequiredRequestsKeys extends keyof Requests<TRequestsData>
  > = Core.RequiredKeys<Requests<TRequestsData>, TRequiredRequestsKeys>
}
```