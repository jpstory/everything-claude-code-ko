# 공통 패턴

## API Response 형식

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

## Custom Hook 패턴

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

## Repository 패턴

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```

## Skeleton 프로젝트

새 기능 구현 시:
1. 실전 검증된 skeleton 프로젝트 검색
2. 병렬 agent로 옵션 평가:
   - 보안 평가
   - 확장성 분석
   - 관련성 점수
   - 구현 계획
3. 가장 적합한 것을 기반으로 clone
4. 검증된 구조 내에서 반복
