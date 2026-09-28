# product-service – Cache chi tiết sản phẩm bằng Redis

- `@Cacheable(value="products", key="#id")`: cache theo `product_id`, key Redis `products::<id>`
- TTL **30 phút** (`app.cache.products-ttl=30m`)
- `@CacheEvict(value="products", key="#id")` khi admin cập nhật / xoá sản phẩm

## Chạy
```bash
docker compose up -d
mvn spring-boot:run
redis-cli FLUSHALL
```

## Kiểm chứng tốc độ
```bash
# Lần 1: Cache MISS -> có SQL, ~200ms
curl -s -o /dev/null -w "%{time_total}s\n" http://localhost:8080/api/products/1
# Lần 2+: Cache HIT -> không SQL, <10ms
curl -s -o /dev/null -w "%{time_total}s\n" http://localhost:8080/api/products/1

redis-cli TTL products::1        # ~1800 giây
```

## Kiểm chứng xoá cache khi admin cập nhật
```bash
curl -X PUT http://localhost:8080/api/products/1 -H "Content-Type: application/json" \
     -d '{"name":"Sản phẩm 1 (bản mới)","imageUrl":"https://cdn.example.com/new.jpg"}'
redis-cli KEYS "products::*"     # products::1 đã biến mất
curl http://localhost:8080/api/products/1   # SQL xuất hiện lại, trả dữ liệu mới
```

> `app.simulate-db-latency-ms=200` giả lập DB chậm khi tải cao (máy local truy vấn theo khoá chính
> thường chỉ vài ms). Đặt `0` để đo tốc độ DB thật.
