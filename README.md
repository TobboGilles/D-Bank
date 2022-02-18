# D-Bank

// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

import "./Token.sol";

contract dbank {
    
Token private token;
   
   address payable public user;
   
    uint depositDuration;
    
    uint interestPersecond;
   
    uint interest;
    
    uint userBlance;
   
   mapping (address => uint ) etherBalanceOf;
   
   mapping (address=> uint ) depositStart;
   
   mapping (address=> bool) isDeposited;
   
   mapping (address => bool) isBorrow;
   
   mapping (address => uint ) CollateralEther;
   
    
   event deposit (address indexed user, uint etherAmount , uint timeStart);
   
   event withdrawal (address indexed, uint etherAmount , uint DepositDuration, uint  interest);
   
   event Borrowal (address indexed, uint etherAmount, uint amount);
   
    constructor (Token _token)  {
        
    token =_token;
        
        }
    
    
    function depoisit () payable public {
        
     // make sure the depoisit has already been done 
     require (isDeposited[msg.sender]= false, "you already deposited");
    
     // deposit should be greater than 0.01ETH
     require (msg.value >= 1e16, "deposit amount should be grater than 0.01ETH");
     
       // increase msg.sender deposit balance 
        
        etherBalanceOf[msg.sender] =  etherBalanceOf[msg.sender]+ msg.value;
        
        //update the holding time of the deposit 
        
        depositStart[msg.sender] = depositStart[msg.sender] + block.timestamp;
        
       // update deposited status to true
        isDeposited[msg.sender] =true; 
        
        emit deposit (msg.sender, msg.value, block.timestamp);
        
        
    }
    
  
  function withdraw (address payable _beneficiaire)  public {
       // make sure user deposited something 
       require (isDeposited[msg.sender]==true, "you have not deposited anything");
        
        userBlance = etherBalanceOf[msg.sender];
        
        // check user hold time 
 
  depositDuration = block.timestamp - depositStart[msg.sender]; 
 
  // send the money back to user with token
  
  
   interestPersecond = 31668017 * (etherBalanceOf[msg.sender] / 1e16);
   
   interest = interestPersecond * depositDuration;
  
   _beneficiaire.transfer(etherBalanceOf[msg.sender]);
   
   // mint the interest of a directtly to the user 
      token.mint(_beneficiaire, interest);
  
  // reset the depositer data 
      etherBalanceOf[msg.sender] = 0;
      
      depositStart[msg.sender] = 0; // set deposit time to zero
      
      isDeposited[msg.sender] = false; // reset deposit ti zero
      
      emit withdrawal (msg.sender, userBlance, depositDuration, interest);
  
    }
    
    
    function Borrow () payable public {
        // Collateral must be >= 0.01 ETH"
        require (msg.value >= 1e16 , "Collateral must be >= 0.01 ETH");
        // make sure the user has not already take loan
        require (isBorrow[msg.sender]== false, "You have already Borrow");
        
        CollateralEther[msg.sender] =  CollateralEther[msg.sender] + msg.value;
        
       // mint 50% of the CollateralEther (user can borrow 50% of what its collateral
        uint TokentoMint = CollateralEther[msg.sender] / 2;
        
        // mint token to borrow
        token.mint (msg.sender, TokentoMint);
        
       // update status borrow to true
        isBorrow[msg.sender] = true;
        
        emit Borrow (msg.sender, CollateralEther[msg.sender], TokentoMint);
    }
    
    
    function payOff () payable public {
        
        require (isBorrow[msg.sender] == true, "You have not borrow any money");
        
        require (token.transferFrom(msg.sender, address(this), CollateralEther[msg.sender]/2  ));
        
        // calculate the fee that the plateform will take 
        uint fee = CollateralEther[msg.sender]/10 ;
        
        // transfer the money back to user minus fee 
        msg.sender.transfer(CollateralEther[msg.sender] - fee);
    
      // reset user's ether balance to zero    
        CollateralEther[msg.sender] = 0;
        
        isBorrow[msg.sender] = false;
        
    }
    
}
