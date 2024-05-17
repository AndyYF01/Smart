Contrato

    // SPDX-License-Identifier: MIT
    pragma solidity >=0.8.17;

    contract SistemaCompraVenta{
    struct vendedor{
        uint256 num;
        string cuidad;
        string nombre;
        uint balance;
    }

    struct comprador{
        uint256 num;
        string nombre;
        uint balance;
    }

    struct producto{
        uint256 num;
        string nombre;
        string unidad_medida;
        uint precio;
        uint cantidad;
    }
    
    struct venta{
        uint256 num_pro;
        uint256 num_com;
    }

    mapping(address => vendedor) public vendedores;
    mapping(address => comprador) public compradores;
    mapping(address => producto[]) public productos;
    mapping(address => venta[]) public ventas;

    constructor(){
        vendedores[0x5B38Da6a701c568545dCfcB03FcB875f56beddC4] = vendedor(1,"moscu","Ivanov Ivan Ivanovich", 1000);
        vendedores[0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2] = vendedor(2,"san petersburgo","Fedorov Fedor Fedorovich", 300);
        
        productos[0x5B38Da6a701c568545dCfcB03FcB875f56beddC4].push(producto(1,"multibarca","unidad",4,3));
        productos[0x5B38Da6a701c568545dCfcB03FcB875f56beddC4].push(producto(2,"celular","unidad",6,7));

        productos[0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2].push(producto(1,"secadora","unidad",2,5));
        productos[0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2].push(producto(2,"plancha","unidad",1,1));

        compradores[0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db] = comprador(1,"artemov artemovich", 50);
        compradores[0x78731D3Ca6b7E34aC0F824c42a7cC18A495cabaB] = comprador(2,"Egor egorovich", 70);
    }

    function comprar_producto(address direccion,uint256 num,uint num2)public iscomprador{
        uint256 index = buscar_producto(num,direccion);
        require(productos[msg.sender][index].num ==num ,"NO ESTA ESTE PRODUCTO");
        ventas[msg.sender].push(venta(num,num2));
        compradores[msg.sender].balance -= productos[direccion][index].precio*num2;
        productos[direccion][index].cantidad -= num2;

    }
    function buscar_producto(uint256 id,address direccion)internal view returns(uint256){
        for(uint i=0;i<productos[direccion].length;i++){
            if(productos[direccion][i].num == id ){
                return i;
            }
        }
        revert('Producto no encontrado');
    }

    modifier isvendedor{
        require(msg.sender==0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 || msg.sender==0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2,"Solo un vendedor puede agregar");
        _;
    }

    modifier iscomprador{
        require(msg.sender==0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db || msg.sender==0x78731D3Ca6b7E34aC0F824c42a7cC18A495cabaB,"Solo un comprador puede comprar");
        _;
    }
    function agregar_producto(uint256 num, string memory nombre,string memory unidad_medida, uint precio, uint cantidad)public isvendedor{
        productos[msg.sender].push(producto(num,nombre,unidad_medida,precio,cantidad));
    }
    
}


    /////////////////

    // SPDX-License-Identifier: GPL-3.0

    pragma solidity ^0.8.17;

    contract Shop {
    struct Product {
        string info;
        uint256 price; 
    }

    struct Crate {
        uint256 shopId;
        address customer;
        bool status;
        uint256[] productsInCrate;
        uint256[] refunds;
        uint256[] boughtProducts; 
    }

    struct Store {
        address owner;
        uint256[] productsInShop; 
    }

    address public admin;
    mapping(uint256 => Product) public products;
    mapping(uint256 => Crate) public crates;
    mapping(uint256 => Store) public shops;

    constructor() {
        admin = msg.sender;
        createProduct(1, "Default Product", 1);
    }

    modifier isAdmin() {
        require(msg.sender == admin, "You are not admin");
        _;
    }

    modifier onlyShopOwner(uint256 shopId) {
        require(msg.sender == shops[shopId].owner, "You are not the shop owner");
        _;
    }

    function createProduct(uint256 productId, string memory _info, uint256 _price) public isAdmin {
        products[productId] = Product(_info, _price);
    }

    function createShop(uint256 shopId, address _owner) public isAdmin {
        uint256[] memory productsInShop;
        shops[shopId] = Store(_owner, productsInShop);
    }

    function addProductToShop(uint256 shopId, uint256 productId) public onlyShopOwner(shopId) {
        shops[shopId].productsInShop.push(productId);
    }

    function createCrate(uint256 shopId, uint256 crateId, address _customer) public {
        require(msg.sender != admin, "Admin can't buy things!");
        require(crates[crateId].status == false, "Crate already exists!");
        uint256[] memory productsInCrate;
        uint256[] memory refunds;
        uint256[] memory boughtProducts;
        crates[crateId] = Crate(shopId, _customer, true, productsInCrate, refunds, boughtProducts);
    }

    function addToCrate(uint256 productId, uint256 crateId) public {
        require(crates[crateId].customer == msg.sender, "You are not the customer!");
        crates[crateId].productsInCrate.push(productId);
    }

    function buyInCrate(uint256 crateId) public payable {
        require(crates[crateId].productsInCrate.length > 0, "Crate is empty!");
        require(msg.sender == crates[crateId].customer, "Not your crate!");

        uint256 totalCost = 0;
        for (uint256 i = 0; i < crates[crateId].productsInCrate.length; i++) {
            uint256 productId = crates[crateId].productsInCrate[i];
            totalCost += products[productId].price;
        }

        require(msg.value >= totalCost, "Insufficient payment");

        for (uint256 i = 0; i < crates[crateId].productsInCrate.length; i++) {
            crates[crateId].boughtProducts.push(crates[crateId].productsInCrate[i]);
        }

        uint256 shopId = crates[crateId].shopId;
        address shopOwner = shops[shopId].owner;

        for (uint256 i = 0; i < crates[crateId].boughtProducts.length; i++) {
            for (uint256 j = 0; j < shops[shopId].productsInShop.length; j++) {
                if (crates[crateId].boughtProducts[i] == shops[shopId].productsInShop[j]) {
                    shops[shopId].productsInShop[j] = shops[shopId].productsInShop[shops[shopId].productsInShop.length - 1];
                    shops[shopId].productsInShop.pop();
                    break;
                }
            }
        }

        delete crates[crateId].productsInCrate;

        payable(shopOwner).transfer(totalCost);
    }

    function refundAttempt(uint256 crateId, uint256 prodId) public {
        require(crates[crateId].customer == msg.sender, "You are not the customer!");
        crates[crateId].refunds.push(prodId);
    }

    function acceptRefund(uint256 crateId, uint256 productId) public payable {
        require(msg.sender == shops[crates[crateId].shopId].owner, "You are not the owner!");

        uint256 price = products[productId].price;
        uint256 shopId = crates[crateId].shopId;

        for (uint256 i = 0; i < crates[crateId].boughtProducts.length; i++) {
            if (crates[crateId].boughtProducts[i] == productId) {
                for (uint256 j = i; j < crates[crateId].boughtProducts.length - 1; j++) {
                    crates[crateId].boughtProducts[j] = crates[crateId].boughtProducts[j + 1];
                }
                crates[crateId].boughtProducts.pop();
                break;
            }
        }

        shops[shopId].productsInShop.push(productId);
        payable(crates[crateId].customer).transfer(price);
    }

    function refuseRefund(uint256 crateId, uint256 productId) public {
        require(msg.sender == shops[crates[crateId].shopId].owner, "You are not the owner!");

        for (uint256 i = 0; i < crates[crateId].refunds.length; i++) {
            if (crates[crateId].refunds[i] == productId) {
                for (uint256 j = i; j < crates[crateId].refunds.length - 1; j++) {
                    crates[crateId].refunds[j] = crates[crateId].refunds[j + 1];
                }
                crates[crateId].refunds.pop();
                break;
            }
        }
    }

    function getCrateList(uint256 crateId) public view returns (uint256[] memory) {
        return crates[crateId].productsInCrate;
    }
}









