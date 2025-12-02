# IMC
import sqlite3
from datetime import datetime
class Database:
    def __init__(self, db_name="imc.db"):
        self.db_name = db_name
        self._create_table()

    def connect(self):
        return sqlite3.connect(self.db_name)

    def _create_table(self):
        conn = self.connect()
        cursor = conn.cursor()

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS registros (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nome TEXT NOT NULL,
                peso REAL NOT NULL,
                altura REAL NOT NULL,
                imc REAL NOT NULL,
                classificacao TEXT NOT NULL,
                data_hora TEXT NOT NULL
            )
        """)

        conn.commit()
        conn.close()

    def salvar_registro(self, nome, peso, altura, imc, classificacao):
        conn = self.connect()
        cursor = conn.cursor()

        data_hora = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        cursor.execute("""
            INSERT INTO registros (nome, peso, altura, imc, classificacao, data_hora)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (nome, peso, altura, imc, classificacao, data_hora))

        conn.commit()
        conn.close()

    def obter_todos(self):
        conn = self.connect()
        cursor = conn.cursor()

        cursor.execute("SELECT * FROM registros ORDER BY id DESC")
        resultados = cursor.fetchall()

        conn.close()
        return resultados

class IMCApp:
    def __init__(self):
        self.db = Database()

    def calcular_imc(self, peso, altura):
        return peso / (altura ** 2)

    def classificar(self, imc):
        if imc < 18.5:
            return "Magreza"
        elif imc < 25:
            return "Normal"
        elif imc < 30:
            return "Sobrepeso"
        elif imc < 35:
            return "Obesidade I"
        elif imc < 40:
            return "Obesidade II"
        else:
            return "Obesidade III"

    def novo_calculo(self):
        nome = input("Nome: ").strip()
        peso = float(input("Peso (kg): "))
        altura = float(input("Altura (m): "))

        imc = self.calcular_imc(peso, altura)
        classificacao = self.classificar(imc)

        self.db.salvar_registro(nome, peso, altura, imc, classificacao)

        print("\n--- Resultado do IMC ---")
        print(f"IMC: {imc:.2f}")
        print(f"Classificação: {classificacao}")
        print("------------------------\n")

    def mostrar_historico(self):
        registros = self.db.obter_todos()

        if not registros:
            print("\nNenhum registro encontrado.\n")
            return

        print("\n====== HISTÓRICO DE IMC ======")
        for r in registros:
            print(f"ID: {r[0]}")
            print(f"Nome: {r[1]}")
            print(f"Peso: {r[2]} kg")
            print(f"Altura: {r[3]} m")
            print(f"IMC: {r[4]:.2f}")
            print(f"Classificação: {r[5]}")
            print(f"Data/Hora: {r[6]}")
            print("-----------------------------")
        print()

    def menu(self):
        while True:
            print("===== APLICAÇÃO DE IMC =====")
            print("1 - Novo cálculo de IMC")
            print("2 - Ver histórico")
            print("0 - Sair")

            opcao = input("Escolha: ").strip()

            if opcao == "1":
                self.novo_calculo()
            elif opcao == "2":
                self.mostrar_historico()
            elif opcao == "0":
                print("Saindo...")
                break
            else:
                print("Opção inválida!\n")

if __name__ == "__main__":
    app = IMCApp()
    app.menu()
