CREATE TABLE Cliente
(
IDCliente int primary key identity,
Nome varchar(50) not null,
Email varchar(50) not null
)

CREATE TABLE Produto
(
IDProduto int primary key identity,
Nome varchar(50) not null,
Preco decimal(10,2) not null
)

CREATE TABLE Pedido
(
IDPedido int primary key identity(1,1),
IDCliente int references Cliente(IDCliente),
DataPedido date not null,
Status varchar(20) check (Status IN ('Aprovado', 'Enviado', 'Cancelado'))
)

CREATE TABLE ItemPedido
(
IDItem int primary key identity(1,1),
IDPedido int references Pedido(IDPedido),
IDProduto int references Produto(IDProduto),
Quantidade int not null,
PrecoUnitario decimal (10,2),
)
 

insert into Cliente (Nome,Email) values
('Pedro Aguiar','pedro@gmail.com'),
('Priscila Gomes','priscila@gmail.com'),
('Matheus Henrique','matheus@gmail.com'),
('maykon Lobo','maykon@gmail.com'),
('Mayara Vicente','mayara@gmail.com')


insert into Produto(Nome,Preco) values
('Monitor Gamer','699.99'),
('Cadeira Gamer','899.99'),
('Mouse ','279.99'),
('Teclado Mecânico','499.90'),
('Headset','699.99')

insert into Pedido (IDCliente,DataPedido, Status) values 
('1','2024/01/16','Enviado'),
('2','2024/02/25','Enviado'),
('3','2024/05/04','Cancelado'),
('4','2024/07/09','Aprovado')



insert into ItemPedido(IDPedido,IDProduto,Quantidade,PrecoUnitario) Values
('21','1','2','699.99'),
('22','2','3','899.99'),
('22','3','1','279.90')

CREATE VIEW vw_PedidosDetalhados AS
SELECT
    c.Nome AS NomeCliente,
    p.DataPedido,
    pr.Nome AS NomeProduto,
    ip.Quantidade,
    ip.PrecoUnitario,
    (ip.Quantidade * ip.PrecoUnitario) AS ValorTotal,
    p.Status
FROM Pedido p
INNER JOIN Cliente c ON p.IDCliente = c.IDCliente
INNER JOIN ItemPedido ip ON p.IDPedido = ip.IDPedido
INNER JOIN Produto pr ON ip.IDProduto = pr.IDProduto;


create function fn_AplicaDesconto 
(

@valor DECIMAL,
@percent INT

)

returns decimal (10,2)
as 
begin 
    declare @ValorComDesconto decimal(10,2)

     set @ValorComDesconto = @Valor - (@valor * @percent/100);

    return @ValorComDesconto
end;

select dbo.fn_AplicaDesconto(699.99,10) as ValorComDesconto

begin tran;
insert into Pedido (IDCliente,DataPedido, Status) values 
('5',getdate(),'Aprovado');
declare @NovoPedido int = scope_identity();

insert into ItemPedido (IDPedido,IDProduto,Quantidade,PrecoUnitario) values
(@NovoPedido,'4','2','499.90');

commit;

select NomeProduto,
sum (Quantidade) as TotalVendido
from vw_PedidosDetalhados
group by NomeProduto
order by TotalVendido desc;

select 
sum (ValorTotal) as FaturamentoTotal
from vw_PedidosDetalhados
