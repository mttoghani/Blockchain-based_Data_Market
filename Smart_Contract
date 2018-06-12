pragma solidity ^0.4.24;

contract Health {
    
    //***********************************************
    //****************** variables ******************
    //***********************************************
    address public admin;
    mapping (address => bool) public doctor;
    mapping (address => bytes32) public patient;
    mapping (address => address) public cure_permission;
    mapping (bytes32 => bytes32) public record;
    mapping (bytes32 => address) public record_owner;
    
    
    //***********************************************
    //****************** events *********************
    //***********************************************
    
    event Register_Doctor(address the_doctor);
    event Register_Patient(address the_patient, bytes32 new_hash);
    event New_Record(address the_patient, bytes32 last_hash, bytes32 new_hash, address the_doctor);
    event Trade_Request(address smartcontract, address buyer);
    
    
    //***********************************************
    //****************** modifiers ******************
    //***********************************************
    
    modifier is_admin() {
        require(msg.sender == admin);
        _;
    }
    
    modifier is_doctor() {
        require(doctor[msg.sender] == true);
        _;
    }
    
    modifier is_registered_patient() {
        require(!(patient[msg.sender]==bytes32(0)));
        _;
    }
    
    
    //*************************************************
    //****************** Constructor ******************
    //*************************************************
    
    constructor() public {
        admin = msg.sender;
    }
    
    
    //***********************************************
    //****************** functions ******************
    //***********************************************
    
    function add_doctors(address[] new_doctors) public is_admin() {
        for (uint8 i = 0; i < new_doctors.length; i++) {
            doctor[new_doctors[i]] = true;
            emit Register_Doctor(new_doctors[i]);
        }
    }
    
    function register_Patient(address new_patient, bytes32 info_hash) public is_admin() {
        require(patient[new_patient]==bytes32(0));
        patient[new_patient] = info_hash;
        emit Register_Patient(new_patient, info_hash);
    }
    
    function allow_doctor(address a_doctor) public is_registered_patient(){
        require(doctor[a_doctor]);
        cure_permission[msg.sender] = a_doctor;
    }
    
    function input_new_tx(address a_patient, bytes32 new_hash) public is_doctor() {
        require(cure_permission[a_patient]==msg.sender);
        cure_permission[a_patient] = address(0);
        emit New_Record(a_patient, patient[a_patient], new_hash, msg.sender);
        record[new_hash] = patient[a_patient];
        patient[a_patient] = new_hash;
        record_owner[new_hash] = a_patient;
    }
    
    function getAdmin() public view returns(address){
        return admin;
    }
    
    function trade_request(address buyer) public{
        emit Trade_Request(msg.sender, buyer);
    }
    
    
    function valid_request(address seller, bytes32 data_hash) public view returns(bool){
        return (record_owner[data_hash]==seller);
    }
    
    
}

////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////

contract Trade {
    
    //***********************************************
    //****************** variables ******************
    //***********************************************
    
    struct Record{
        address owner;
        bytes32 enc_hash;
        bytes32 key;
        uint seller_value;
        uint buyer_value;
        uint time;
        uint8 state;
    }
    
    //***********************************************
    //****************** variables ******************
    //***********************************************
    
    Health h;
    address public buyer;
    address public admin;
    bool public valid;
    mapping (bytes32 => Record) public records;
    
    
    //***********************************************
    //****************** events *********************
    //***********************************************
    
    event New_Trade(address smartcontract);
    event New_Request(bytes32 data_hash, bytes32 enc_hash, uint seller_deposit);
    event Buyer_Confirmation(bytes32 data_hash, uint buyer_deposit);
    event Send_Key(bytes32 data_hash, bytes32 the_key);
    
    //***********************************************
    //****************** modifiers ******************
    //***********************************************
    
    modifier is_buyer() {
        require(msg.sender == buyer);
        _;
    }
    
    modifier is_admin() {
        require(msg.sender == admin);
        _;
    }
    
    modifier is_valid() {
        require(valid == true);
        _;
    }
    
    //*************************************************
    //****************** Constructor ******************
    //*************************************************
    
    constructor(address health) public {
        buyer = msg.sender;
        h = Health(health);
        admin = h.getAdmin();
        h.trade_request(buyer);
    }
    
    
    //***********************************************
    //****************** functions ******************
    //***********************************************
    function Validate() public is_admin{
        valid = true;
        emit New_Trade(this);
    }
    
    function seller_request(bytes32 data_hash, bytes32 enc_hash) payable public is_valid{
        require(h.valid_request(msg.sender, data_hash));
        require(records[data_hash].owner==address(0));
        records[data_hash] = Record({owner:msg.sender, enc_hash:enc_hash,
                                    key:bytes32(0), seller_value:msg.value,
                                    buyer_value:uint(0), time:now, state:0});
                                    
        emit New_Request(data_hash, enc_hash, msg.value);
    }
    
    function buyer_confirmation(bytes32 data_hash) payable public is_buyer is_valid{
        require(records[data_hash].owner!=address(0));
        require(records[data_hash].state==0);
        records[data_hash].state = 1;
        records[data_hash].time = now;
        records[data_hash].buyer_value = msg.value;
        emit Buyer_Confirmation(data_hash, msg.value);
    }
    
    function send_key(bytes32 data_hash, bytes32 the_key) public is_valid{
        require(msg.sender==records[data_hash].owner);
        require(records[data_hash].state==1);
        records[data_hash].state = 2;
        records[data_hash].key=the_key;
        records[data_hash].time = now;
        emit Send_Key(data_hash, the_key);
    }
    
    function dispute(bytes32 data_hash, bytes32 IV, bytes enc_data, bytes32[] enc_hashes, bytes32[] data_hashes, uint order) public is_buyer is_valid{
        require((records[data_hash].state==2) && (now < records[data_hash].time + 120 minutes));
        bytes32 goal = sha256(abi.encodePacked(IV, enc_data));
        uint temp = order;
        for (uint8 i = 0; i < enc_hashes.length; i++) {
            if(temp%2 == 0){
                goal = sha256(abi.encodePacked(goal, enc_hashes[i]));
            }
            else{
                goal = sha256(abi.encodePacked(enc_hashes[i], goal));
            }
            temp = temp/2;
        }
        if(goal == records[data_hash].enc_hash){
            goal = data_hashes[0];
            temp = order;
            for (i = 1; i < data_hashes.length; i++) {
                if(temp%2 == 0){
                    goal = sha256(abi.encodePacked(goal, data_hashes[i]));
                }
                else{
                    goal = sha256(abi.encodePacked(data_hashes[i], goal));
                }
                temp = temp/2;
            }
            if(goal == data_hash){
                goal = sha256(decrypt(IV, enc_data, records[data_hash].key, "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"));
                if(goal != data_hashes[0]){
                    buyer.transfer((records[data_hash].seller_value + records[data_hash].buyer_value));
                    records[data_hash].state = 3;
                    records[data_hash].seller_value = 0;
                    records[data_hash].buyer_value = 0;  
                }
                else{
                    records[data_hash].owner.transfer((records[data_hash].seller_value + records[data_hash].buyer_value));
                    records[data_hash].state = 4;
                    records[data_hash].seller_value = 0;
                    records[data_hash].buyer_value = 0;        
                }
            }
            else{
            records[data_hash].owner.transfer((records[data_hash].seller_value + records[data_hash].buyer_value));
            records[data_hash].state = 4;
            records[data_hash].seller_value = 0;
            records[data_hash].buyer_value = 0;
            }
        }
        else{
            records[data_hash].owner.transfer((records[data_hash].seller_value + records[data_hash].buyer_value));
            records[data_hash].state = 4;
            records[data_hash].seller_value = 0;
            records[data_hash].buyer_value = 0;
        }
    }
    
    function claim_money(bytes32 data_hash) payable public is_valid{
        require(records[data_hash].owner!=address(0));
        require(    ((records[data_hash].state==2) && (msg.sender==records[data_hash].owner) && (now > records[data_hash].time + 120 minutes))
                 || ((records[data_hash].state==1) && (msg.sender==buyer || msg.sender==records[data_hash].owner) && (now > records[data_hash].time + 5 minutes ))
                 || ((records[data_hash].state==0) && (msg.sender==records[data_hash].owner) && (now > records[data_hash].time + 5 minutes))
                );
        if(records[data_hash].state==2){
            records[data_hash].owner.transfer((records[data_hash].seller_value + records[data_hash].buyer_value));
            records[data_hash].state = 5;
            records[data_hash].seller_value = 0;
            records[data_hash].buyer_value = 0;
        }
        else if(records[data_hash].state==1){
            records[data_hash].owner.transfer(records[data_hash].seller_value);
            buyer.transfer(records[data_hash].buyer_value);
            records[data_hash] = Record({owner:address(0), enc_hash:bytes32(0), key:bytes32(0), seller_value:uint(0), buyer_value:uint(0), time:uint(0), state:0});
        }
        else{
            records[data_hash].owner.transfer(records[data_hash].seller_value);
            records[data_hash] = Record({owner:address(0), enc_hash:bytes32(0), key:bytes32(0), seller_value:uint(0), buyer_value:uint(0), time:uint(0), state:0});
        }
    }
    
    function decrypt(bytes32 IV, bytes enc_data, bytes32 key, bytes temp)public returns(bytes){
        bytes32 tempb;
        uint8 tempu;
        uint i;
        uint j;
        for(j = 0; j < 32; j++){
            temp[32+j] = key[j];
        }
        for(j = 0; j < 32; j++){
            temp[j] = IV[j];
        }
        for(i = 0; i < enc_data.length; i++){
            for(j = 0; j < 63; j++){
                temp[63-j] = temp[62-j];
            }
            temp[0] = bytes1(uint8(temp[3]) ^ uint8(temp[37]) ^ uint8(temp[59]));
            tempb = sha256(temp);
            tempu = uint8(tempb[7]) ^ uint8(tempb[17]) ^ uint8(tempb[29]);
            enc_data[i] = bytes1(uint8(enc_data[i]) ^ tempu);
        }
        return (enc_data);
    }


}
