# ABB-e-ndices-de-SBD
Trabalho de Estruturas de Dados : ABB e índices de SBD

from typing import Optional, List


class Registro:
    def __init__(self, cpf: str, nome: str, data_nasc: str):
        self.cpf = cpf
        self.nome = nome
        self.data_nasc = data_nasc
        self.deletado = False  # Flag para marcar exclusão

    def __lt__(self, outro):
        return self.cpf < outro.cpf

    def __str__(self):
        return f"CPF: {self.cpf}, Nome: {self.nome}, Nasc: {self.data_nasc}, Deletado: {self.deletado}"


class NoArvore:
    def __init__(self, registro: Registro, pos: int):
        self.registro = registro
        self.pos = pos  # Índice na lista EDL
        self.esq: Optional[NoArvore] = None
        self.dir: Optional[NoArvore] = None


class ArvoreBinariaBusca:
    def __init__(self):
        self.raiz: Optional[NoArvore] = None

    def inserir(self, registro: Registro, pos: int):
        def _inserir(no: Optional[NoArvore], reg: Registro, pos: int) -> NoArvore:
            if no is None:
                return NoArvore(reg, pos)
            if reg < no.registro:
                no.esq = _inserir(no.esq, reg, pos)
            else:
                no.dir = _inserir(no.dir, reg, pos)
            return no

        self.raiz = _inserir(self.raiz, registro, pos)

    def buscar(self, cpf: str) -> Optional[NoArvore]:
        def _buscar(no: Optional[NoArvore], cpf: str) -> Optional[NoArvore]:
            if no is None or no.registro.cpf == cpf:
                return no
            if cpf < no.registro.cpf:
                return _buscar(no.esq, cpf)
            else:
                return _buscar(no.dir, cpf)

        return _buscar(self.raiz, cpf)

    def remover(self, cpf: str):
        def _remover(no: Optional[NoArvore], cpf: str) -> Optional[NoArvore]:
            if no is None:
                return None
            if cpf < no.registro.cpf:
                no.esq = _remover(no.esq, cpf)
            elif cpf > no.registro.cpf:
                no.dir = _remover(no.dir, cpf)
            else:
                if no.esq is None:
                    return no.dir
                elif no.dir is None:
                    return no.esq
                # Encontrar substituto (mínimo da direita)
                min_dir = no.dir
                while min_dir.esq:
                    min_dir = min_dir.esq
                no.registro = min_dir.registro
                no.pos = min_dir.pos
                no.dir = _remover(no.dir, min_dir.registro.cpf)
            return no

        self.raiz = _remover(self.raiz, cpf)

    def percorrer_in_ordem(self, funcao_visita):
        def _in_ordem(no: Optional[NoArvore]):
            if no:
                _in_ordem(no.esq)
                funcao_visita(no)
                _in_ordem(no.dir)

        _in_ordem(self.raiz)

    def percorrer_pre_ordem(self, funcao_visita):
        def _pre_ordem(no: Optional[NoArvore]):
            if no:
                funcao_visita(no)
                _pre_ordem(no.esq)
                _pre_ordem(no.dir)

        _pre_ordem(self.raiz)

    def percorrer_pos_ordem(self, funcao_visita):
        def _pos_ordem(no: Optional[NoArvore]):
            if no:
                _pos_ordem(no.esq)
                _pos_ordem(no.dir)
                funcao_visita(no)

        _pos_ordem(self.raiz)


# ========== Simulação do EDL (lista de registros) ==========
if __name__ == "__main__":
    edl: List[Registro] = []
    abb = ArvoreBinariaBusca()


    def inserir_registro(cpf: str, nome: str, data: str):
        reg = Registro(cpf, nome, data)
        edl.append(reg)
        pos = len(edl) - 1
        abb.inserir(reg, pos)


    def buscar_registro(cpf: str):
        no = abb.buscar(cpf)
        if no is None:
            print(f"Registro com CPF {cpf} não encontrado.")
        else:
            reg = edl[no.pos]
            if reg.deletado:
                print(f"Registro com CPF {cpf} foi deletado.")
            else:
                print(f"Registro encontrado: {reg}")


    def remover_registro(cpf: str):
        no = abb.buscar(cpf)
        if no:
            edl[no.pos].deletado = True
            abb.remover(cpf)
            print(f"Registro com CPF {cpf} marcado como deletado e removido da ABB.")
        else:
            print(f"Registro com CPF {cpf} não encontrado para remover.")


    def criar_edl_ordenado():
        edl_ordenado: List[Registro] = []

        def adicionar(no: NoArvore):
            edl_ordenado.append(edl[no.pos])

        abb.percorrer_in_ordem(adicionar)
        return edl_ordenado


    # ========== Teste rápido ==========
    inserir_registro("111", "Ana", "1990-01-01")
    inserir_registro("222", "Bruno", "1985-02-02")
    inserir_registro("333", "Carlos", "1992-03-03")

    print("\n--- Buscar antes de deletar ---")
    buscar_registro("222")

    print("\n--- Remover Bruno ---")
    remover_registro("222")

    print("\n--- Buscar depois de deletar ---")
    buscar_registro("222")

    print("\n--- EDL ordenado ---")
    edl_ord = criar_edl_ordenado()
    for reg in edl_ord:
        if not reg.deletado:
            print(reg)
