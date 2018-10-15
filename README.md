# node-projects-home-biz

sample biz rule test
biz rule engine

bizrule_engine.js
// Test ftn with .call
// this rule engine, takes rules (array of rules, int startid, int endruleid)
// process each rules and return, which rules have been processed and if fail, return fail message

var BizRuleEngine = function(rules){ //, startRuleId, endRuleId){
	this.rules = rules;
	this.pathRef = []; // send back which rules are processed and status (t/f)
};

BizRuleEngine.prototype.execute = function(data, ruleSequences, callback){
	console.log('rule engine is executing...');
	var _rules = _.clone(this.rules);
	var _data = _.clone(data);
	var _pathRef = _.clone(this.pathRef); // send back which rules are processed and status (t/f)	
	var _ruleSequences = _.clone(ruleSequences);
	var _numOfRuleSeq = _ruleSequences.length; 	
	// process all the rules, until end of the rule id indecator present
	// local ftn
	function getRule(id){
		for(var j=0; j<_rules.length; j++){
			if(_rules[j].id === id){
				return _rules[j];
			}
		}
	}

	function getRuleSequence(id){
		for(var m=0; m<_ruleSequences.length; m++){
			if (_ruleSequences[m].id === id){
				return _ruleSequences[m];
			}
		}
	}

	var _ruleSequence = _ruleSequences[0];
	var _ruleId = _ruleSequences[0].id;

	for(k=0; k<_numOfRuleSeq; k++){
		_ruleSequence = getRuleSequence(_ruleId);
		var _ruleMain = getRule(_ruleId);
		console.log(_ruleSequence);
		console.log(_ruleMain);

		if (_ruleMain == undefined){
			console.log('ERROR-RULES UNDEFINED FOR RULE ID ' + _ruleId);
			return false;
		}
		if (_ruleSequence == undefined){
			console.log('ERROR-RULE SEQUENCE UNDEFINED FOR RULE ID ' + _ruleId);
			return false;
		}

		var _rule = _ruleMain.rule;
		// evaluate rule when
		var _when = _rule.when;
		var _onSuccess = _rule.onSuccess;
		var _onFail = _rule.onFail;
		//console.log(_rule);

		var isPassed = _when.call(data);
		var respObj = isPassed ? _onSuccess.call(data) : _onFail.call(data);
		//console.log(respObj);

		 var _pathRefData = { 
		 	"id": _ruleId, 
		 	"name": _ruleMain.name,
		 	"status": isPassed,
		 	"data": respObj
		 };
		_pathRef.push(_pathRefData);
		//console.log(_pathRef);

		// get next rule id
		_ruleId = isPassed ? _ruleSequence.idOnSuccess : _ruleSequence.idOnFail;
		console.log('next id=' + _ruleId + ' k=' + k);

		if (_ruleId <= 0){
			console.log("END OF PROCESSING OF RULES");
			break;
		}
	}

	_data.refData = _pathRef;
	callback(_data);
}
------------------------------------------------------

bizrules.js
// create biz rules and send it array..
// create an object and expose a method, so rules will be contained within obj, 
// else rules will be global (not good)

var BizRulesCreditCardApproval = function(){

	// get the labels from constant file
	var constants = new ConstantsCreditcard();

	// age must be over 18
	var r0 = {
		"id": 100,
		"name": constants.ageGt18,
		"rule": {
			"when": function(){
				console.log('100-when');
				return (this.age > 18);
			},
			"onSuccess": function(){ 
				console.log('100-success');
				return {"message": "Age is > 10." }; 
			},
			"onFail": function(){
				console.log('100-fail');
				return {"message": "Age must be greater than 18." }; 
			}
		}
	};

	// income is greater than minimum expected?
	var r1 = {
		"id": 101,
		"name": constants.incomeValidator,
		"rule": {
			"when": function(){
				console.log('101-when');
				return (this.income > 30000);
			},
			"onSuccess": function(){ 
				console.log('101-success');
				return {"message": "Income approved" }; 
			},
			"onFail": function(){
				console.log('101-fail');
				return {"message": "Not enough income, but validate for property and mtg", "score": 0 }; 
			}
		}
	};

	// property value is greater than minimum expected?
	var r2 = {
		"id": 102,
		"name": constants.propertyValidator,
		"rule": {
			"when": function(){
				console.log('102-when');
				return (this.propertyValue > 500000);
			},
			"onSuccess": function(){ 
				console.log('102-success');
				return {"message": "Property approved"}; 
			},
			"onFail": function(){ 
				console.log('102-fail');
				return {"message": "Not enough propertyValue" }; 
			}
		}
	};

	// mortgage is rating
	var r3 = {
		"id": 103,
		"name": constants.mortgageValidator,
		"rule": {
			"when": function(){
				console.log('103-when');
				return (this.mortgage >= 0);
			},
			"onSuccess": function(){ 
				console.log('103-success');
				return {"message": "mortgage approved" }; 
			},
			"onFail": function(){
				console.log('103-fail');
				return {"message": "Too much mortgage" }; 
			}
		}
	};


	this.rules = [r0, r1, r2, r3];
	// set start and end rule ids
	//this.startRuleId = 100;
	//this.endRuleId = 999;
};

BizRulesCreditCardApproval.prototype.getRules = function(){
	return { 
		"rules": this.rules
		//"startRuleId": this.startRuleId,
		//"endRuleId": this.endRuleId 
	}
}









/*
var BizRulesCreditCardApproval = function(){

	// get the labels from constant file
	var constants = new ConstantsCreditcard();

	// age must be over 18
	var r0 = {
		"id": 100,
		"name": constants.ageGt18,
		"rule": {
			"when": function(){
				console.log('100-when');
				return (this.age > 18);
			},
			"onSuccess": function(){ 
				console.log('100-success');
				return {"id": 101, "message": "Age is > 10." }; 
			},
			"onFail": function(){
				console.log('100-fail');
				return {"id": 999, "message": "Age must be greater than 18." }; 
			}
		}
	};

	// income is greater than minimum expected?
	var r1 = {
		"id": 101,
		"name": constants.incomeValidator,
		"rule": {
			"when": function(){
				console.log('101-when');
				return (this.income > 3000);
			},
			"onSuccess": function(){ 
				console.log('101-success');
				return {"id": 102, "message": "Income approved" }; 
			},
			"onFail": function(){
				console.log('100-fail');
				return {"id": 102, "message": "Not enough income, but validate for property and mtg", "score": 0 }; 
			}
		}
	};

	// property value is greater than minimum expected?
	var r2 = {
		"id": 102,
		"name": constants.propertyValidator,
		"rule": {
			"when": function(){
				console.log('102-when');
				return (this.propertyValue > 500000);
			},
			"onSuccess": function(){ 
				console.log('102-success');
				return {"id": 103, "message": "Property approved"}; 
			},
			"onFail": function(){ 
				console.log('102-fail');
				return {"id": 999, "message": "Not enough propertyValue" }; 
			}
		}
	};

	// mortgage is rating
	var r3 = {
		"id": 103,
		"name": constants.mortgageValidator,
		"rule": {
			"when": function(){
				console.log('103-when');
				return (this.mortgage >= 0);
			},
			"onSuccess": function(){ 
				console.log('103-success');
				return {"id": 999, "message": "mortgage approved" }; 
			},
			"onFail": function(){
				console.log('103-fail');
				return {"id": 999, "message": "Too much mortgage" }; 
			}
		}
	};


	this.rules = [r0, r1]; //, r2, r3];
	// set start and end rule ids
	this.startRuleId = 100;
	this.endRuleId = 999;
};

BizRulesCreditCardApproval.prototype.getRules = function(){
	return { 
		"rules": this.rules, 
		"startRuleId": this.startRuleId,
		"endRuleId": this.endRuleId 
	}
}

*/


-------------------------
constants.js
var ConstantsCreditcard = function(){
	this.ageGt18 = "Age greater than 18."; 
	this.incomeValidator = "Income validator"; 
	this.propertyValidator = "Property validator";  
	this.mortgageValidator = "Mortgage validator";
  
  ---------------------------------------
  
  
ext_biz_evalution_ftn.js
function evaluateIncome(income){
	// evaluate income. if less than 3K return false..
	if (income < 3000){
		return false;
	}
	return true;
}


// return score based on income
function getIncomeScoreOnSuccess(income){
	var score = 0;
	if (income > 100000){
		score = 1000;
	}
	else if (income > 80000){
		score = 800;
	}
	else if (income > 50000){
		score = 500;
	}
	else if (income > 20000){
		score = 200;
	}
	else if (income > 3000){
		score = 100;
	}
	else {
		score = 0;
	}

	return score;
}


// property 
function evaluatePropertyValue(pv){
	if (pv > 0){
		return true;
	}
	return false;
}

function getPropertyValueScoreOnSuccess(pv){
	var score = 0;
	if (pv > 500000){
		score = 2000;
	}
	else if (pv > 200000){
		score = 1000;
	}
	else if (pv > 100000){
		score = 500;
	}
	else {
		score = 0;
	}
	return score;
}



// mtg
function getMortgateScoreOnSuccess(pv, mtg){
	var mtgPerct = mtg / pv;
	console.log('mtgPerct=' + mtgPerct);
	var score = 0;
	if (mtgPerct <= 0){
		score = 2000;
	}
	else if (mtgPerct <= 0.5){
		score = 1000;
	}
	else if (mtgPerct <= 0.75){
		score = 500;
	} else {
		score = 0;
	}
	return score;
}


-------------------------------------
test_bizrules_cr_validation.js


// test biz rules for credit card validation
console.log('test biz rules for credit card validation');

// get biz rules. an array containing rules
var bizRulesCreditCardApproval = new BizRulesCreditCardApproval();
var ruleObj = bizRulesCreditCardApproval.getRules();
//console.log(ruleObj);

var rules = ruleObj.rules;   // array of rules


// create biz rule engine
var bizRuleEngine = new BizRuleEngine(rules);             //, startRuleId, endRuleId);
console.log(bizRuleEngine);


// rules sequence
var ruleSeqs = [
	{"id": 100, "idOnSuccess": 101, "idOnFail": 0},
	{"id": 101, "idOnSuccess": 102, "idOnFail": 102},
	{"id": 102, "idOnSuccess": 103, "idOnFail": 0},
	{"id": 103, "idOnSuccess": 0, "idOnFail": 0}
];

// fact
var fact = { "name": "James Jonathan", "age": 40, "income": 90000, "propertyValue": 800000, "mortgage": 3000 };
// execute the rules, by providing the fact
bizRuleEngine.execute(fact,ruleSeqs, mycallback);
//------------------------------------------------------------------------------


// fact
var fact = { "name": "James22 Jonathan22", "age": 40, "income": 20000, "propertyValue": 800000, "mortgage": 3000 };
// execute the rules, by providing the fact
////bizRuleEngine.execute(fact,ruleSeqs, mycallback);


// test 2 differt sequence
var ruleSeqs22 = [
	{"id": 100, "idOnSuccess": 103, "idOnFail": 0},
	{"id": 103, "idOnSuccess": 0, "idOnFail": 0}
];

////bizRuleEngine.execute(fact,ruleSeqs22, mycallback);
//------------------------------------------------------------------------------





// test 3 differt sequence, in REVERSE
var ruleSeqs33 = [
	{"id": 103, "idOnSuccess": 101, "idOnFail": 0},
	{"id": 101, "idOnSuccess": 100, "idOnFail": 0},
	{"id": 100, "idOnSuccess": 0, "idOnFail": 0}
];

bizRuleEngine.execute(fact,ruleSeqs33, mycallback);


function mycallback(data){
	console.log('in callback. rule evaluation done...');
	console.log(data);
	//console.log('total score');
	var score = 0;
	data.refData.forEach(function(element){
		//console.log(element.data);
		//console.log(element.data.score);
		//score += element.data.score;
	});
}







// ========================================================================================
// test for customer validations
// next set of biz rules

// get biz rules. an array containing rules
var bizRulesCustomerPromotions = new BizRulesCustomerPromotions();
var ruleObjCustPromo = bizRulesCustomerPromotions.getRules();
//console.log(ruleObj);

var rulesCustPromo = ruleObjCustPromo.rules;   // array of rules


// create biz rule engine
var bizRuleEngineCustPromo = new BizRuleEngine(rulesCustPromo);             //, startRuleId, endRuleId);
console.log(bizRuleEngineCustPromo);


// rules sequence
var ruleSeqsCustPromo = [
	{"id": 1, "idOnSuccess": 2, "idOnFail": 3},
	{"id": 2, "idOnSuccess": 0, "idOnFail": 0},
	{"id": 3, "idOnSuccess": 4, "idOnFail": 0},
	{"id": 4, "idOnSuccess": 0, "idOnFail": 0}
];

// fact
var factCust = { "name": "James Jonathan", "customerType": "GOLD", "totalSales": 7000 };
// execute the rules, by providing the fact
bizRuleEngineCustPromo.execute(factCust, ruleSeqsCustPromo, mycallback);


var factCust22 = { "name": "James22 Jonathan22", "customerType": "SILVER", "totalSales": 17000 };
// execute the rules, by providing the fact
bizRuleEngineCustPromo.execute(factCust22, ruleSeqsCustPromo, mycallback);













