# eth-event

[![Pypi Status](https://img.shields.io/pypi/v/eth-event.svg)](https://pypi.org/project/eth-event/) [![Build Status](https://img.shields.io/travis/com/iamdefinitelyahuman/eth-event.svg)](https://travis-ci.com/iamdefinitelyahuman/eth-event) [![Coverage Status](https://coveralls.io/repos/github/iamdefinitelyahuman/eth-event/badge.svg?branch=master)](https://coveralls.io/github/iamdefinitelyahuman/eth-event?branch=master)

Simple tools for Ethereum event decoding and topic generation.

## Installation

You can install the latest release via ``pip``:

```bash
$ pip install eth-brownie
```

Or clone the repository and use ``setuptools`` for the most up-to-date version:

```bash
$ python3 setup.py install
```

## Usage

The package includes five functions:

* `get_topics(abi)`: Given a contract ABI, returns a dictionary of `{'event name': "encrypted topic"}`.

```python
>>> import eth_event

>>> abi = [{'name': 'Approval', 'anonymous': False, 'type': 'event', 'inputs': [{'name': 'owner', 'type': 'address', 'indexed': True}, {'name': 'spender', 'type': 'address', 'indexed': True}, {'name': 'value', 'type': 'uint256', 'indexed': False}]}, {'name': 'Transfer', 'anonymous': False, 'type': 'event', 'inputs': [{'name': 'from', 'type': 'address', 'indexed': True}, {'name': 'to', 'type': 'address', 'indexed': True}, {'name': 'value', 'type': 'uint256', 'indexed': False}]}]

>>> eth_event.get_topics(abi)
{'Transfer': '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef', 'Approval': '0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925'}
```

* `get_event_abi(abi)`: Given a contract ABI, returns a dictionary of `{'encrypted topic': "ABI"}`. Useful for stripping an ABI so that only event related data remains.

```python
>>> event_abi = eth_event.get_event_abi(abi)
{'0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef': {'name': 'Transfer', 'inputs': [{'name': 'from', 'type': 'address', 'indexed': True}, {'name': 'to', 'type': 'address', 'indexed': True}, {'name': 'value', 'type': 'uint256', 'indexed': False}]}, '0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925': {'name': 'Approval', 'inputs': [{'name': 'owner', 'type': 'address', 'indexed': True}, {'name': 'spender', 'type': 'address', 'indexed': True}, {'name': 'value', 'type': 'uint256', 'indexed': False}]}}
```

* `decode_event(event, abi)`: Given a single event from a transaction log and an ABI, returns the decoded event. The ABI may supplied in the normal contract format, or as a dictionary value generated by `get_event_abi`

```python
>>> tx = token.transfer(account[1], 100, {'from': account[0]})
<Transaction object '0xd0b9f575747eafdce3ba6ee8d6c16146bfff35cb86bec0a1909ab04fa94fc024'>

>>> log = tx.logs[0]
>>> eth_event.decode_event(log, token.abi)
{'name': 'Transfer', 'data': [{'name': 'from', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a'}, {'name': 'to', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a', 'decoded': True}, {'name': 'tokens', 'type': 'uint256', 'value': 100, 'decoded': True}]}

>>> eth_event.decode_event(log, event_abi[log['topics'][0]])
{'name': 'Transfer', 'data': [{'name': 'from', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a'}, {'name': 'to', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a', 'decoded': True}, {'name': 'tokens', 'type': 'uint256', 'value': 100, 'decoded': True}]}
```

* `decode_logs(logs, abi)`: Given an entire transaction log and an ABI, returns the decoded events. The ABI may supplied in the normal contract format, or as a dictionary value generated by `get_event_abi`

```python
>>> tx = token.transfer(account[1], 100, {'from': account[0]})
<Transaction object '0x615a157e84715d5f960a38fe2a3ddb566c8393cfc71f15b06170a0eff74dfdde'>

>>> eth_event.decode_event(tx.logs, token.abi)
[{'name': 'Transfer', 'data': [{'name': 'from', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a'}, {'name': 'to', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a', 'decoded': True}, {'name': 'tokens', 'type': 'uint256', 'value': 100, 'decoded': True}]}]
```

* `decode_trace(trace, abi)`: Given the `structLog` from a [debug_traceTransaction](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#debug_tracetransaction) RPC call, returns a list of the decoded events. Useful for obtaining events when a transaction reverts.

```python
>>> tx = token.transfer(account[1], 100, {'from': account[0]})
<Transaction object '0x71a7146996957b8a465a4e5dd41951d614254f8271fc9140b875c4fc55dde578'>

>>> trace = web3.providers[0].make_request("debug_traceTransaction", ['0x71a7146996957b8a465a4e5dd41951d614254f8271fc9140b875c4fc55dde578', {}])['result']['structLogs]
]
>>> eth_event.decode_trace(trace, token.abi)
[{'name': 'Transfer', 'data': [{'name': 'from', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a'}, {'name': 'to', 'type': 'address', 'value': '0xbd4940951bfa463f8fb6db762e55686f6cfdb73a', 'decoded': True}, {'name': 'tokens', 'type': 'uint256', 'value': 100, 'decoded': True}]}]
```

## Limitations

If an array is indexed in an event, the topic is generated as a sha3 hash and so cannot be decrypted. In this case, the unencrypted topcic is returned and `decoded` is set to `False`.

## Development

This project is still in development and should be considered an alpha. Comments, questions, criticisms and pull requests are welcomed.

## License

This project is licensed under the [MIT license](LICENSE).
