# Rust/Go 고성능 엔진 이식 설계도 🦀🐹

## 📌 개요
Python 기반 퀀트/뉴스 엔진의 병목 구간을 Rust/Go로 이식하여 성능 향상을 목표로 함.

## 🔍 병목 구간 분석

| 모듈 | 현재 언어 | 병목 원인 | 이식 대상 |
|------|----------|----------|----------|
| `quant_swarm_engine.py` | Python | 병렬 연산 시 GIL 제한 | ✅ Rust |
| `news_factory_v1.py` | Python | RSS 파싱 I/O 대기 | ✅ Go |
| `dart_monitor.py` | Python | API 호출 + JSON 파싱 | ⚠️ 검토 |
| `unified_status_checker.py` | Python | 다중 API 호출 | ⚠️ 검토 |

## 🦀 Rust 이식 대상 (CPU 집약적)

### 1. 퀀트 스웜 엔진
```rust
// quant_swarm_engine.rs
use rayon::prelude::*;

fn parallel_backtest(params: Vec<Param>) -> Vec<Result> {
    params.par_iter()
        .map(|p| run_backtest(p))
        .collect()
}
```

**예상 성능 향상**: 5~10배 (GIL 해제 + 네이티브 병렬)

### 2. 백테스트 연산
- 현재: pandas + numpy (Python)
- 목표: polars-rs 또는 직접 구현

## 🐹 Go 이식 대상 (I/O 집약적)

### 1. 뉴스 팩토리 파이프라인
```go
// news_factory.go
func fetchRSSFeeds(urls []string) []News {
    var wg sync.WaitGroup
    results := make(chan News, len(urls))
    
    for _, url := range urls {
        wg.Add(1)
        go func(u string) {
            defer wg.Done()
            results <- parseRSS(u)
        }(url)
    }
    
    wg.Wait()
    close(results)
    return collect(results)
}
```

**예상 성능 향상**: 3~5배 (goroutine 병렬 I/O)

### 2. KIND RSS 스캐너
- 현재: feedparser (Python)
- 목표: Go의 gofeed 라이브러리

## 📅 이식 로드맵

| 단계 | 작업 | 예상 기간 |
|------|------|----------|
| Phase 1 | Go로 뉴스 팩토리 프로토타입 | 1주 |
| Phase 2 | Rust로 스웜 엔진 프로토타입 | 2주 |
| Phase 3 | Python 바인딩 (PyO3/CGO) | 1주 |
| Phase 4 | 통합 테스트 및 배포 | 1주 |

## 🎯 목표 KPI
- 뉴스 수집 속도: 현재 5초 → 1초 이하
- 백테스트 연산: 현재 30초 → 5초 이하
- 메모리 사용량: 50% 감소

---
*피터린치 🐭 | 2026-02-02 작성*
