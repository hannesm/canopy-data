---
title: Better errors and logging
author: talex5 (Thomas Leonard)
abstract: Improved connect reporting and logging ready for testing
---

I've been making lots of (API breaking) changes to Mirage to get proper error messages and logging.

To try it, pin my opam remote and get the updated mirage-skeleton:

    opam remote add talex5 'https://github.com/talex5/mirage-dev.git#better-errors'
    opam upgrade
    git clone -b better-errors https://github.com/talex5/mirage-skeleton.git

You should notice several differences:

- The example `config.ml` files mostly have `with_mirage_logs` wrappers. This enables output of log messages from early boot (previously, you could only enable logging after boot finished).
- The network stack no longer takes a `CONSOLE` argument. It uses the Logs library instead.
- The generated `main.ml` files now have much better error handling. In particular:
  - Errors are handled next to the `connect` call, since this is where we know how to display the error.
  - Functions that never returned errors no longer return result types.
  - Many libraries provide a `pp_error` function to display formatted error messages.

## Examples

### Block device error

Before:

    Fatal error: exception Failure("block11")

After:

    *** Setup failure ***

    Error connecting block device "disk.img":
      block-unix: connecting "/home/user/work/cubieboard/skeleton/block/disk.img":
      Unix.Unix_error(Unix.ENOENT, "open", "/home/user/work/cubieboard/skeleton/block/disk.img")

### Network device error

Before:

    Fatal error: exception Failure("net11")

After:

    *** Setup failure ***

    Error connecting network device "tap0":
      undiagnosed error - Failure("tun[TUNSETIFF]: Operation not permitted")

## Example config.ml changes

Before:

    open Mirage

    let main = foreign "Unikernel.Main" (console @-> stackv4 @-> job)

    let stack = generic_stackv4 default_console tap0

    let () =
      register "network" [
        main $ default_console $ stack
      ]

After (stack no longer takes a console and logging is enabled):

    open Mirage

    let main = foreign "Unikernel.Main" (console @-> stackv4 @-> job)

    let stack = generic_stackv4 tap0

    let () =
      register "network" [
	with_mirage_logs (main $ default_console $ stack)
      ]


## Generated main.ml changes

Before (error messages `_e` being ignored):

    module Unikernel1 = Unikernel.Main(Console_unix)(Block)

    let argv_unix1 = lazy (
      OS.Env.argv () >>= (fun x -> Lwt.return (`Ok x))
      )

    let console_unix_01 = lazy (
      Console_unix.connect "0"
      )

    let block11 = lazy (
      Block.connect "/src/block/disk.img"
      )

    let key1 = lazy (
      let __argv_unix1 = Lazy.force argv_unix1 in
      __argv_unix1 >>= function
      | `Error _e -> fail (Failure "argv_unix1")
      | `Ok _argv_unix1 ->
      return (Functoria_runtime.with_argv Key_gen.runtime_keys "block_test" _argv_unix1)
      )

    let f11 = lazy (
      let __console_unix_01 = Lazy.force console_unix_01 in
      let __block11 = Lazy.force block11 in
      __console_unix_01 >>= function
      | `Error _e -> fail (Failure "console_unix_01")
      | `Ok _console_unix_01 ->
      __block11 >>= function
      | `Error _e -> fail (Failure "block11")
      | `Ok _block11 ->
      Unikernel1.start _console_unix_01 _block11 >>= fun t -> Lwt.return (`Ok t)
      )

After (unnecessary errors removed, necessary ones reported, logging configured):

    module Unikernel1 = Unikernel.Main(Block) 
     
    module Mirage_logs1 = Mirage_logs.Make(Clock) 
     
    let block11 = lazy ( 
      Block.connect "/src/mirage-skeleton/block/disk.img" >|= function 
      | `Error e -> die "@[<v 2>Error connecting block device %S:@ %a@]" "disk.img" Block.pp_error e 
      | `Ok x -> x 
      ) 
     
    let argv_unix1 = lazy ( 
      OS.Env.argv () 
      ) 
     
    let clock1 = lazy ( 
      return () 
      ) 
     
    let f11 = lazy ( 
      let __block11 = Lazy.force block11 in 
      __block11 >>= fun _block11 -> 
      Unikernel1.start _block11 
      ) 
     
    let key1 = lazy ( 
      let __argv_unix1 = Lazy.force argv_unix1 in 
      __argv_unix1 >>= fun _argv_unix1 -> 
      return (Functoria_runtime.with_argv Key_gen.runtime_keys "block_test" _argv_unix1) 
      ) 
     
    let mirage_logs1 = lazy ( 
      Logs.set_level (Some Logs.Info); 
      Mirage_logs1.(create () |> run) @@ fun () -> 
      let __clock1 = Lazy.force clock1 in 
      let __f11 = Lazy.force f11 in 
      __clock1 >>= fun _clock1 -> 
      __f11 >>= fun _f11 -> 
      return (_clock1, _f11) 
      ) 

Logging:

    $ sudo ./mir-network
    Netif: plugging into tap0 with mac c2:9d:56:19:d7:2c
    Netif: connect tap0
    2016-03-15 11:40.40: INF [tcpip-stack-direct] Manager: connect
    2016-03-15 11:40.40: INF [tcpip-stack-direct] Manager: configuring
    2016-03-15 11:40.40: INF [tcpip-stack-direct] Manager: Interface to 10.0.0.2 nm 255.255.255.0 gw [10.0.0.1]
    2016-03-15 11:40.40: INF [arpv4] ARP: sending gratuitous from 10.0.0.2
    2016-03-15 11:40.40: INF [tcpip-stack-direct] Manager: configuration done

(still need to add logging to `Netif` though)

# Next

There's still plenty more to do. More libraries need logging, the console devices don't yet report useful error information, and there's plenty more bad error handling after the `connect` functions are fixed. Let me know if you want to help out...

