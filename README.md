# CLASSES
Função extra
Vocês devem ter notado que a função para validar telefone possui uma dependência, que é a fncRecupera_Nummeros.

RETURNS BIT
AS 
BEGIN
 
    DECLARE @chr CHAR, @tamanho INT
    
    -- Verifica se possui 8 caracteres
    IF (LEN(@Nr_Cep) < 8)
        RETURN 0
        
    WHILE (LEN(@Nr_Cep) > 0)
    BEGIN
    
        SELECT @tamanho = LEN(@Nr_Cep), @chr = LEFT(@Nr_Cep,1)
        
        -- Verifica se o número informado possui apenas números
        IF CHARINDEX(@chr,'0123456789') = 0
        BEGIN
            RETURN 0
            BREAK
        END
        
        SET @Nr_Cep = STUFF(@Nr_Cep,1,1,'') -- retira o primeiro dígito
        
    END
    
    RETURN 1
    
END
CREATE FUNCTION dbo.fncValidarEmail(@Ds_Email varchar(max))
RETURNS BIT
AS BEGIN
 
    DECLARE @Retorno BIT = 0
 
    SELECT @Retorno = 1
    WHERE @Ds_Email NOT LIKE '%[^a-z,0-9,@,.,_,-]%'
    AND @Ds_Email LIKE '%_@_%_.__%'
    AND @Ds_Email NOT LIKE '%_@@_%_.__%'
 
    RETURN @Retorno
 
END
CREATE FUNCTION [dbo].[fncValida_CPF](
    @Nr_Documento VARCHAR(11)
)
RETURNS BIT -- 1 = válido, 0 = inválido
WITH SCHEMABINDING
BEGIN
 
    DECLARE
        @Contador_1 INT,
        @Contador_2 INT,
        @Digito_1 INT,
        @Digito_2 INT,
        @Nr_Documento_Aux VARCHAR(11)
 
    -- Remove espaços em branco
    SET @Nr_Documento_Aux = LTRIM(RTRIM(@Nr_Documento))
    SET @Digito_1 = 0
 
 
    -- Remove os números que funcionam como validação para CPF, pois eles "passam" pela regra de validação
    IF (@Nr_Documento_Aux IN ('00000000000', '11111111111', '22222222222', '33333333333', '44444444444', '55555555555', '66666666666', '77777777777', '88888888888', '99999999999', '12345678909'))
        RETURN 0
 
 
    -- Verifica se possui apenas 11 caracteres
    IF (LEN(@Nr_Documento_Aux) <> 11)
        RETURN 0
    ELSE 
    BEGIN
 
        -- Cálculo do segundo dígito
        SET @Nr_Documento_Aux = SUBSTRING(@Nr_Documento_Aux, 1, 9)
 
        SET @Contador_1 = 2
 
        WHILE (@Contador_1 < = 10)
        BEGIN 
            SET @Digito_1 = @Digito_1 + (@Contador_1 * CAST(SUBSTRING(@Nr_Documento_Aux, 11 - @Contador_1, 1) as int))
            SET @Contador_1 = @Contador_1 + 1
        end 
 
        SET @Digito_1 = @Digito_1 - (@Digito_1/11)*11
 
        IF (@Digito_1 <= 1)
            SET @Digito_1 = 0
        ELSE 
            SET @Digito_1 = 11 - @Digito_1
 
        SET @Nr_Documento_Aux = @Nr_Documento_Aux + CAST(@Digito_1 AS VARCHAR(1))
 
        IF (@Nr_Documento_Aux <> SUBSTRING(@Nr_Documento, 1, 10))
            RETURN 0
        ELSE BEGIN 
        
            -- Cálculo do segundo dígito
            SET @Digito_2 = 0
            SET @Contador_2 = 2
 
            WHILE (@Contador_2 < = 11)
            BEGIN 
                SET @Digito_2 = @Digito_2 + (@Contador_2 * CAST(SUBSTRING(@Nr_Documento_Aux, 12 - @Contador_2, 1) AS INT))
                SET @Contador_2 = @Contador_2 + 1
            end 
 
            SET @Digito_2 = @Digito_2 - (@Digito_2/11)*11
 
            IF (@Digito_2 < 2)
                SET @Digito_2 = 0
            ELSE 
                SET @Digito_2 = 11 - @Digito_2
 
            SET @Nr_Documento_Aux = @Nr_Documento_Aux + CAST(@Digito_2 AS VARCHAR(1))
 
            IF (@Nr_Documento_Aux <> @Nr_Documento)
                RETURN 0
                
        END
    END 
    
    RETURN 1
    
END
CREATE FUNCTION [dbo].[fncValida_CNPJ] ( @CNPJ VARCHAR(14) )
RETURNS BIT
AS
BEGIN
 
    DECLARE
        @INDICE INT,
        @SOMA INT,
        @DIG1 INT,
        @DIG2 INT,
        @VAR1 INT,
        @VAR2 INT,
        @RESULTADO CHAR(1)
 
 
    SET @SOMA = 0
    SET @INDICE = 1
    SET @RESULTADO = 0
    SET @VAR1 = 5 /* 1a Parte do Algorítimo começando de "5" */
 
    WHILE ( @INDICE < = 4 )
    BEGIN
        SET @SOMA = @SOMA + CONVERT(INT, SUBSTRING(@CNPJ, @INDICE, 1)) * @VAR1
        SET @INDICE = @INDICE + 1 /* Navegando um-a-um até < = 4, as quatro primeira posições */
        SET @VAR1 = @VAR1 - 1       /* subtraindo o algorítimo de 5 até 2 */
    END
 
       
    SET @VAR2 = 9
    WHILE ( @INDICE <= 12 )
    BEGIN
        SET @SOMA = @SOMA + CONVERT(INT, SUBSTRING(@CNPJ, @INDICE, 1)) * @VAR2
        SET @INDICE = @INDICE + 1
        SET @VAR2 = @VAR2 - 1            
    END
 
    SET @DIG1 = ( @SOMA % 11 )
 
 
   /* SE O RESTO DA DIVISÃO FOR < 2, O DIGITO = 0 */
    IF @DIG1 < 2
        SET @DIG1 = 0;
    ELSE /* SE O RESTO DA DIVISÃO NÃO FOR < 2*/
        SET @DIG1 = 11 - ( @SOMA % 11 );
 
 
    SET @INDICE = 1
    SET @SOMA = 0
    SET @VAR1 = 6 /* 2a Parte do Algorítimo começando de "6" */
    SET @RESULTADO = 0
 
 
    WHILE ( @INDICE <= 5 )
    BEGIN
        SET @SOMA = @SOMA + CONVERT(INT, SUBSTRING(@CNPJ, @INDICE, 1)) * @VAR1
        SET @INDICE = @INDICE + 1 /* Navegando um-a-um até < = 5, as quatro primeira posições */
        SET @VAR1 = @VAR1 - 1       /* subtraindo o algorítimo de 6 até 2 */
    END
 
 
 
    /* CÁLCULO DA 2ª PARTE DO ALGORÍTIOM 98765432 */
    SET @VAR2 = 9
    WHILE ( @INDICE <= 13 )
    BEGIN
        SET @SOMA = @SOMA + CONVERT(INT, SUBSTRING(@CNPJ, @INDICE, 1)) * @VAR2
        SET @INDICE = @INDICE + 1
        SET @VAR2 = @VAR2 - 1            
    END
 
 
    SET @DIG2 = ( @SOMA % 11 )
 
 
   /* SE O RESTO DA DIVISÃO FOR < 2, O DIGITO = 0 */
 
    IF @DIG2 < 2
        SET @DIG2 = 0;
 
    ELSE /* SE O RESTO DA DIVISÃO NÃO FOR < 2*/
        SET @DIG2 = 11 - ( @SOMA % 11 );
 
 
    IF ( @DIG1 = SUBSTRING(@CNPJ, LEN(@CNPJ) - 1, 1) ) AND ( @DIG2 = SUBSTRING(@CNPJ, LEN(@CNPJ), 1) )
        SET @RESULTADO = 1
    ELSE
        SET @RESULTADO = 0
 
 
 
    RETURN @RESULTADO
 
 
END
CREATE FUNCTION [dbo].[fncValida_Documento] ( 
    @Nr_Documento VARCHAR(14) 
)
RETURNS BIT
AS BEGIN
 
    DECLARE @Retorno BIT = 0
 
 
    IF (LEN(@Nr_Documento) = 11)
    BEGIN
 
        -- Valida CPF
        IF (@Nr_Documento IN ('00000000000', '11111111111', '22222222222', '33333333333', '44444444444', '55555555555', '66666666666', '77777777777', '88888888888', '99999999999', '12345678909'))
            SET @Retorno = 0
        ELSE
            SET @Retorno = CLR.dbo.fncValida_CPF(@Nr_Documento)
 
    END
    ELSE BEGIN 
        
        -- Valida CNPJ
        IF (LEN(@Nr_Documento) = 14)
            SET @Retorno = CLR.dbo.fncValida_CNPJ(@Nr_Documento)
        ELSE
            SET @Retorno = 0
 
    END
 
 
    RETURN @Retorno
    
 
END
CREATE FUNCTION [dbo].[fncValida_Telefone] (@Nr_Telefone VARCHAR(15))
RETURNS BIT
AS
BEGIN
    
    DECLARE 
        @Retorno BIT = 1,
        @Nr_Telefone_Valida VARCHAR(15)
 
 
    -- Verifica se possui caracteres não numéricos
    SET @Nr_Telefone_Valida = dbo.fncRecupera_Numeros(@Nr_Telefone)
 
    IF (@Nr_Telefone != @Nr_Telefone_Valida)
        SET @Retorno = 0
 
 
    -- Verifica a quantidade de digitos
    SET @Nr_Telefone_Valida = (CASE 
                                    WHEN LEN(@Nr_Telefone) = 8 THEN @Nr_Telefone
                                    WHEN LEN(@Nr_Telefone) = 9 THEN @Nr_Telefone
                                    WHEN LEN(@Nr_Telefone) = 10 THEN RIGHT(@Nr_Telefone,8)
                                    WHEN LEN(@Nr_Telefone) = 11 THEN RIGHT(@Nr_Telefone,9) 
                                    ELSE NULL
                               END)
 
 
    -- Verifica se possui apenas números repetidos
    IF(RIGHT(@Nr_Telefone_Valida, 8) IN ('99999999','88888888','77777777','66666666','55555555','44444444','33333333','22222222','11111111','00000000'))
        SET @Retorno = 0 
        
        
    -- Verifica se é string vazia
    IF (@Nr_Telefone_Valida IS NULL)
        SET @Retorno = 0                            
 
 
    RETURN @Retorno
 
    
END
CREATE FUNCTION [dbo].[fncRecupera_Numeros] ( @str VARCHAR(500) )
RETURNS VARCHAR(500)
BEGIN  
 
    DECLARE @startingIndex INT  
    SET @startingIndex = 0  
    
    WHILE (1 = 1)
    BEGIN  
    
        SET @startingIndex = PATINDEX('%[^0-9]%', @str)  
        IF @startingIndex <> 0
            SET @str = REPLACE(@str, SUBSTRING(@str, @startingIndex, 1), '')  
        ELSE
            BREAK
            
    END  
    
    
    RETURN @str  
    
    
END


Resumo:
Validação de CEP
Essa é uma validação simples, que apenas verifica se a quantidade de caracteres está correta e se todos a string informada possui 8 caracteres numéricos. Para uma validação mais forte, sugiro que seja adquirido uma base junto aos Correios e a função de validação deverá realizar uma consulta nessa base para confirmar os dados e a veracidade do CEP.
Validação de CPF
Essa função irá validar a quantidade de dígitos do CPF (precisa ter 11), validar se o CPF informado não está na lista de CPF’s repetidos, mas que “passam” pelas validações padrão e faz o cálculo e validação do CPF informado para identificar se é válido ou não.
Validação de CNPJ
Essa função irá validar a quantidade de dígitos do CNPJ (precisa ter 14) e realizar o cálculo e validação do CNPJ informado para identificar se é válido ou não.
Validação de CPF e CNPJ
Essa função tem por objetivo, realizar a validação de CPF e CNPJ, utilizando as funções acima dependendo da quantidade de caracteres do Nº de documento informado.
Validação de Telefone
Essa função irá verificar a quantidade de dígitos do telefone e se o número informado não é formado apenas por números repetidos.
Função extra
Vocês devem ter notado que a função para validar telefone possui uma dependência, que é a fncRecupera_Numeros. Essa função tem por finalidade, retornar apenas caracteres numéricos (0 a 9) e uma string, removendo strings, acentos, caracteres especiais, etc.
