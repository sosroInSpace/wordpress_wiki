```
/*
  check.js
*/

class SchemaError extends Error {
  constructor(message, key) {
    super(key ? `${key}: ${message}` : message);
    this.name = 'SchemaError';
    this.key = key != null ? key : null;
  }
}

function sErr (message, key) {
  throw new SchemaError(message, key);
}

function check (arg, func, key) {
  func(arg, key);
}

function match (arg, func, key) {
  try {
    func(arg, key);
    return true;
  } catch (err) {
    return false;
  }
}

function getError (arg, func, key) {
  try {
    func(arg, key);
    return '';
  } catch (err) {
    return err.message;
  }
}

/*
  Basic Types.
*/

const IsString = (x, key) => {
  if (typeof x !== 'string') {
    sErr('must be a String', key);
  }
};

const IsObject = (x, key) => {
  if (!x || x.constructor !== Object) {
    sErr('must be an Object', key);
  }
};

const IsArray = (x, key) => {
  if (!Array.isArray(x)) {
    sErr('must be an Array', key);
  }
};

const IsNumber = (x, key) => {
  if (typeof x !== 'number') {
    sErr('must be a Number', key);
  }
};

const IsBoolean = (x, key) => {
  if (typeof x !== 'boolean') {
    sErr('must be a Boolean', key);
  }
};

const IsInteger = (x, key) => {
  if (!Number.isInteger(x)) {
    sErr('must be an Integer', key);
  }
};

const IsNull = (x, key) => {
  if (x !== null) {
    sErr('must be Null', key);
  }
}

const IsUndefined = (x, key) => {
  if (x !== undefined) {
    sErr('must be Undefined', key);
  }
};

const IsFunction = (x, key) => {
  if (typeof x !== 'function') {
    sErr('must be a Function', key);
  }
}

const IsAny = (x, key) => {};

/*
  More complicated types.
*/

const IsRegExpOf = (regExp) => (x, key) => {
  IsString(x, key);
  if (!regExp.test(x)) {
    sErr(`must match ${regExp}`, key);
  }
};

const IsStringOfLength = (min, max) => (x, key) => {
  IsString(x, key);
  if (max === undefined) {
    max = min;
  }
  if (x.length < min || x.length > max) {
    const rangeText = max !== min ? `${min} - ${max}` : min;
    sErr(`must be String of length ${rangeText}`, key);
  }
};

const IsObjectWith = (obj) => (x, key) => {
  IsObject(x, key);
  for (let [k, func] of Object.entries(obj)) {
    func(x[k], key ? `${key}.${k}` : k);
  }
};

const IsObjectExact = (obj) => (x, key) => {
  IsObject(x, key);
  for (let k of Object.keys(x)) {
    if (!obj[k]) {
      sErr('is illegal key', key ? `${key}.${k}` : k);
    }
  }
  IsObjectWith(obj)(x, key);
};

const IsArrayOfLength = (min, max) => (x, key) => {
  IsArray(x, key);
  if (max === undefined) {
    max = min;  // If max is undefined, use min.
  }
  if (x.length < min || x.length > max) {
    const rangeText = max !== min ? `${min} - ${max}` : min;
    sErr(`must be Array of length ${rangeText}`, key);
  }
};

const IsArrayOf = (func) => (x, key) => {
  IsArray(x, key);
  for (let i = 0; i < x.length; i++) {
    func(x[i], key ? `${key}.${i}` : i);
  }
};

const IsArrayOfLengthWith = (min, max, func) => (x, key) => {
  if (func === undefined) {
    func = max;  // If func is undefined, assume min = max.
    max = undefined;
  }
  IsArrayOfLength(min, max)(x, key);
  IsArrayOf(func)(x, key);
};

// Based on: https://github.com/manishsaraan/email-validator/blob/master/index.js
// NOTE: We need international emails to work, so we can't be picky with this.
const IsEmail = (x, key) => {
  IsStringOfLength(0, 256)(x, key);
  
  const [account, address] = x.split('@');
  if (account.length === 0 || account.length > 64) {
    sErr('Invalid email address', key);
  }
  if (address.length === 0) {
    sErr('Invalid email address', key);
  }
  
  const domainParts = address.split('.');
  if (domainParts.some(part => part.length > 63)) {
    sErr('Invalid email address', key);
  }
};

/*
  Utility types.
*/

const IsMaybe = (func) => (x, key) => {
  if (x != null) {
    func(x, key);
  }
};

const IsOptional = (func) => (x, key) => {
  if (x !== undefined) {
    func(x, key);
  }
};

const IsOneOf = (...funcs) => (x, key) => {
  const errors = [];
  for (let func of funcs) {
    try {
      func(x, key);
      return;
    } catch (err) {
      errors.push(err.message);
    }
  }
  sErr(`Failed checks: (${errors.join(', ')})`, key);
};

const IsCustom = (func) => (x, key) => {
  func(x, key);
};

module.exports = {
  check,
  IsString,
  IsObject,
  IsArray,
  IsNumber,
  IsBoolean,
  IsInteger,
  IsNull,
  IsUndefined,
  IsFunction,
  IsAny,
  IsRegExpOf,
  IsStringOfLength,
  IsObjectWith,
  IsObjectExact,
  IsArrayOfLength,
  IsArrayOf,
  IsArrayOfLengthWith,
  IsEmail,
  IsMaybe,
  IsOptional,
  IsOneOf,
  IsCustom
};

```