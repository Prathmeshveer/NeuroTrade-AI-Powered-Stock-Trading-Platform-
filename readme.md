.\venv\Scripts\Activate.ps1
venv\Scripts\activate
(Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned) ; (& d:\Stock\venv\Scripts\Activate.ps1)

cd ml-service
python -m uvicorn api_server:app --reload --port 8000


cd frontend
npm start