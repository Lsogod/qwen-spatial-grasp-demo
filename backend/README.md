# Backend

`vision_agent_v2.py` 的增强链路依赖这里的 FastAPI 服务。

启动方式：

```bash
uvicorn app:app --host 0.0.0.0 --port 8000
```

主要接口：

- `POST /vision/sam`
- `POST /vision/dino`
- `POST /vision/dino_trace`
