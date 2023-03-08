// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 < 0.9.0;
pragma experimental ABIEncoderV2;

contract _CORRALONES_ {

    // Direccion de la OMS -> Owner / Dueño del contrato
    address public CORRALONES;

    // Constructor del contenido
    constructor () public {
        CORRALONES = msg.sender;
    }

   // Mapping para relacionar los empresas grua (direccion/address) con la validez del sistema de gestion
   mapping (address => bool) public Validacion_EmpresaGrua;

   // Relacionar una direccion de una empresa grua con su contrato
   mapping (address => address) public EmpresaGrua_Contrato;
   
   // Ejemplo 1: 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 -> true = TIENE PERMISOS PARA CREAR SU SMART CONTRACT
   // Ejemplo 2: 0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2 -> false = NO TIENE PERMISOS PARA CREAR SU SMART CONTRACT

   // Array de direcciones que almacene los contratos de los empresa de grua validados
   address [] public direcciones_EmpresaGrua;

   // Array de las direcciones que soliciten acceso
   address [] public Solicitudes;
  
   // Eventos a 
   event SolicitudAcceso (address);
   event NuevoCentroValidado (address);
   event NuevoContrato (address, address);


   // Modificador que permita unicamente la ejecucion de funciones por la empresa de grua
    modifier UnicamenteCORRALONES(address _direccion) {
       require(_direccion == CORRALONES, "No tienes permisos para realizar esta funcion.");
       _;
    }
   
    // Funcion para solicitar acceso al sistema de carros
    function SolicitarAcceso() public {
        // Almacenar la direccion en el array de solicitudes
        Solicitudes.push(msg.sender);
        // Emision del evento
        emit SolicitudAcceso (msg.sender);
    }

    // Funcion que visualiza las direcciones que han solicitado este acesso
    function VisualizarSolicitudes() public view UnicamenteCORRALONES(msg.sender) returns (address [] memory) {
        return Solicitudes;
    }

    // Funcion para validar nuevos empresa de grua que puedan autogestionarse -> UnicamenteCORRALONES
    function EmpresasGrua (address _empresaGrua) public UnicamenteCORRALONES(msg.sender) {
       // Asignacion del estado de validez al empresa de grua
       Validacion_EmpresaGrua[_empresaGrua] = true;
       // Emision del evento
       emit NuevoCentroValidado(_empresaGrua);
    }
    

    // Funcion que permita crear un contrato inteligente
    function FactoryEmpresaGrua() public {
        //Filtrado para que unicamente los centros de salud validados sean capaces de ejecutar esta funcion
        require (Validacion_EmpresaGrua[msg.sender] == true, "No tienes permisos para ejecutar esta funcion.");
        // Generar un Smart Contract -> Generar su direccion
        address contrato_EmpresaGrua = address (new EmpresaGrua(msg.sender));
        // Almacenamiento la direccion del contrato en el array
        direcciones_EmpresaGrua.push(contrato_EmpresaGrua);
        // Relacion entre Empresa Grua y su contrato
        EmpresaGrua_Contrato[msg.sender] = contrato_EmpresaGrua;
        // Emisio del evento
        emit NuevoContrato(contrato_EmpresaGrua, msg.sender);
    }


}


//Comtrato autogestinable por la empresa grua//
contract EmpresaGrua {

    // Direcciones iniciales
    address public DireccionEmpresaGrua;
    address public DireccionContrato;

    constructor (address _direccion) public {
        DireccionEmpresaGrua = _direccion;
        DireccionContrato = address(this);       
    }
 
    // Mapping que relaciona una ID con un resultado de carro
    mapping (bytes32 => Resultados) ResultadosCARRO;

    // Estructura de los resultados
    struct Resultados {
        bool diagnostico;
        string CodigoFOLIO;
    }

    // Eventos
    event NuevoResultado (bool, string);

    // Filtrar las funciones a ejecutar a las empresas de grua
    modifier UnicamenteEmpresaGrua(address _direccion) {
        require (_direccion == DireccionEmpresaGrua, "No tienes permisos para ejecutar esta funcion.");
        _;
    }

    // Funciones para emitir un resultado de la prueba del carro
    // Formato de los campos de entrada: | 12345X | true | QmbGWKtXivEbxN8q83N78CLHc8dgCaN4ntYpADfwdEMxm3
    function ResultadosPruebaCarro(string memory _idPersona, bool _resultadoCARRO, string memory _codigoFOLIO) public UnicamenteEmpresaGrua(msg.sender){
        // Hash de la identificacion de la persona
        bytes32 hash_idPersona = keccak256 (abi.encodePacked(_idPersona));
        // Relacion del hash de la persona con la estructura de los resultados 
        ResultadosCARRO[hash_idPersona] = Resultados(_resultadoCARRO, _codigoFOLIO);
        // Emision de un evento
        emit NuevoResultado(_resultadoCARRO, _codigoFOLIO);
    }

    // Funcion que permita la visualizacion de los resultados
    function VisualizarResultados(string memory _idPersona) public view returns (string memory _resultadoPrueba, string memory _codigoFOLIO) {
        // Hash de la identidad de la persona
        bytes32 hash_idPersona = keccak256 (abi.encodePacked(_idPersona));
        // Retorno de un booleano como un string
        string memory resultadoPrueba;
        if (ResultadosCARRO[hash_idPersona].diagnostico == true){
            resultadoPrueba = "ENCONTRADO";
        } else{
            resultadoPrueba = "NO ENCONTRADO";
        }
        //Retorno de los parametros necesarios
        _resultadoPrueba = resultadoPrueba;
        _codigoFOLIO = ResultadosCARRO[hash_idPersona].CodigoFOLIO;
    }   


}






