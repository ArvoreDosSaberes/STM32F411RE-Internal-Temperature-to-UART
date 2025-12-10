# Padrões de código

- Usar linguagem C padrão C11 (`CMAKE_C_STANDARD 11`).
- Comentar funções e módulos principais em português, explicando:
  - objetivo
  - parâmetros
  - retorno
  - efeitos colaterais
- Nomear funções de alto nível de forma descritiva, por exemplo
  `read_internal_temperature`, `print_temperature`.
- Evitar literais mágicos: preferir constantes nomeadas quando fizer sentido.
