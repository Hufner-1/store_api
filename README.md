    from fastapi import FastAPI, HTTPException
    from sqlalchemy.exc import IntegrityError
    from typing import Optional
    from fastapi_pagination import Page, paginate
    from fastapi_pagination.ext.sqlalchemy import paginate
    from pydantic import BaseModel
    from sqlalchemy import create_engine, Column, Integer, String
    from sqlalchemy.orm import sessionmaker
    from sqlalchemy.ext.declarative import declarative_base

    app = FastAPI()

    # Configuração do banco de dados
    DATABASE_URL = "mongodb://localhost:27017"
    engine = create_engine(DATABASE_URL)
    SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

    Base = declarative_base()

    class Produto(Base):
    __tablename__ = "produtos"

    id = Column(Integer, primary_key=True, index=True)
    nome = Column(String)
    preco = Column(Integer)
    estoque = Column(Integer)

    Base.metadata.create_all(bind=engine)

    class ProdutoCreate(BaseModel):
    nome: str
    preco: int
    estoque: int

    @app.get("/produtos", response_model=Page[Produto])
    async def read_produtos(nome: Optional[str] = None):
    db = SessionLocal()
    produtos = db.query(Produto)
    if nome:
        produtos = produtos.filter(Produto.nome == nome)
    return paginate(produtos)

    @app.post("/produtos")
    async def create_produto(produto: ProdutoCreate):
    db = SessionLocal()
    try:
        db_produto = Produto(**produto.dict())
        db.add(db_produto)
        db.commit()
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=303, detail="Já existe um produto cadastrado com o nome: x")
    finally:
        db.close()
