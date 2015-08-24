Setup of the example
====================

Two groups (primary groups): `/health/alex/steps` and `/health/alex`

Membership and scheduled access permissions:

- for `/health/alex/steps` group:
  - Alice (`/alice`) and Bob (`/bob`) authorized access of data for the 24hours (=24hour schedule)
  - Yingdi (`/yingdi`) authorized access of data for one hour only (7am-8am)
- for `/health/alex` group
  - Alice (`/alice`) authorized access of data for one hour only (9pm-10pm)

---

Group manager(s) actions:
=========================

## Assumption

Membership is known a priori, e.g., members explicitly gave public keys to the manager.

## Timeline

- *Once a day*, some time before midnight

  - **`/health/alex/steps` group**

    - Generate three E-KEYS/D-KEYS public/private key pairs

      - `/health/alex/steps/E-KEY/<from><today>00:00/<until><today>07:00`
      - `/health/alex/steps/E-KEY/<from><today>07:00/<until><today>08:00`
      - `/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00`

    - Encrypt each of the corresponding D-KEYS with member's public keys

      - **(for Alice and Bob)**

        - `/health/alex/steps/D-KEY/<from><today>00:00/<until><today>07:00/<for>/alice/<version>`
        - `/health/alex/steps/D-KEY/<from><today>00:00/<until><today>07:00/<for>/bob/<version>`
        - `/health/alex/steps/D-KEY/<from><today>07:00/<until><today>08:00/<for>/alice/<version>`
        - `/health/alex/steps/D-KEY/<from><today>07:00/<until><today>08:00/<for>/bob/<version>`
        - `/health/alex/steps/D-KEY/<from><today>08:00/<until><today>24:00/<for>/alice/<version>`
        - `/health/alex/steps/D-KEY/<from><today>08:00/<until><today>24:00/<for>/bob/<version>`
      - **(for Yingdi)**

        - `/health/alex/steps/<D-KEY>/<from><today>07:00/<until><today>08:00/<for>/yingdi/<version>`

  - **`/health/alex` group**

    - Generate one E-KEY/D-KEY public/private key pair

      - `/health/alex/E-KEY/<from><today>21:00/<until><today>22:00`

    - Encrypt the corresponding D-KEY with the users public key

      - `/health/alex/D-KEY/<from><today>21:00/<until><today>22:00/<for>/alice`

---

Producer actions
================

## Assumptions

Producer is configured with a name for samples, e.g., `/health/alex/steps/raw`

After configured, producer assumes that it belongs to groups defined by the name hierarchy:

- `/health/alex/steps/raw`
- `/health/alex/steps`
- `/health/alex`

Groups `/health` and `/` can be omitted for security purposes.

## General timeline

Every hour or some time before an hour boundary:

- For each configured group, fetch E-KEY if there are no existing E-KEYs that cover the next hour

- Generate C-KEY for the next hour

- Encrypt C-KEY with all E-KEYs that cover the next hour

## Example timeline

Assuming there are no prior E-KEYs existed:

- **00:00**

  - Send interests for all groups

            /health/alex/steps/raw/E-KEY/<from><today>00:00 
              |
              +--> timeout or NACK

            /health/alex/steps/E-KEY/<from><today>00:00
              |
              +--> /health/alex/steps/E-KEY/<from><today>00:00/<until><today>07:00

            /health/alex/E-KEY/<from><today>00:00 
              |
              +--> timeout or NACK

  - Generate C-KEY for an hour

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>00:00``

  - Encrypt C-KEY with all matching E-KEYs

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>00:00/<for>/health/alex/steps/E-KEY/<from><today>00:00/<until><today>07:00``

- **01:00**

  - Send interest for groups without covering E-KEY (``/health/alex/steps`` already has matching E-KEY)

            /health/alex/steps/raw/E-KEY/<from><today>01:00 
              |
              +--> timeout or NACK

            /health/alex/E-KEY/<from><today>01:00 
              |
              +--> timeout or NACK

  - Generate C-KEY for an hour

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>01:00``

  - Encrypt C-KEY with all matching E-KEYs

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>01:00/<for>/health/alex/steps/E-KEY/<from><today>00:00/<until><today>07:00``

- **02:00 ... 06:00** (repeat what has been done at **01:00** using an appropriate hour boundary)

- **07:00**

  - Send interests for all groups

            /health/alex/steps/raw/E-KEY/<from><today>07:00 
              |
              +--> timeout or NACK

            /health/alex/steps/E-KEY/<from><today>07:00
              |
              +--> /health/alex/steps/E-KEY/<from><today>07:00/<until><today>08:00

            /health/alex/E-KEY/<from><today>00:00 
              |
              +--> timeout or NACK

  - Generate C-KEY for an hour

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>07:00``

  - Encrypt C-KEY with all matching E-KEYs

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>07:00/<for>/health/alex/steps/E-KEY/<from><today>07:00/<until><today>08:00``

- **08:00**

  - Send interests for all groups

            /health/alex/steps/raw/E-KEY/<from><today>08:00 
              |
              +--> timeout or NACK

            /health/alex/steps/E-KEY/<from><today>08:00
              |
              +--> /health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00

            /health/alex/E-KEY/<from><today>00:00 
              |
              +--> timeout or NACK

  - Generate C-KEY for an hour

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>08:00``

  - Encrypt C-KEY with all matching E-KEYs

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>08:00/<for>/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00``

- **09:00** (repeat what has been done at **01:00** using an appropriate hour boundary)

- **10:00**

  - Send interest for groups without covering E-KEYs (``/health/alex/steps`` already has matching E-KEY)

            /health/alex/steps/raw/E-KEY/<from><today>10:00 
              |
              +--> timeout or NACK

            /health/alex/E-KEY/<from><today>10:00 
              |
              +--> timeout or NACK

  - Generate C-KEY for an hour

    * `/health/alex/steps/C-KEY/<for-hour>/<today>10:00`

  - Encrypt C-KEY with all matching E-KEYs

    * `/health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00`


- **11:00 ... 20:00** (repeat what has been done at **01:00** using an appropriate hour boundary)

- **21:00**

  - Send interest for groups without covering E-KEYs (/health/alex/steps already has matching E-KEY)

            /health/alex/steps/raw/E-KEY/<from><today>21:00 
              |
              +--> timeout or NACK

            /health/alex/E-KEY/<from><today>00:00 
              |
              +--> /health/alex/E-KEY/<from><today>21:00/<until><today>22:00

  - Generate C-KEY for an hour

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>21:00``

  - Encrypt C-KEY with all matching E-KEYs

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>21:00/<for>/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00``

    * ``/health/alex/steps/C-KEY/<for-hour>/<today>21:00/<for>/health/alex/E-KEY/<from><today>21:00/<until><today>22:00``

- **22:00, 23:00** (repeat what has been done at **01:00** using an appropriate hour boundary)

---

Consumer actions
================

Retrieve data item `/health/alex/steps/raw/<today>10:11`:

             +-------------------------------------------------------------------+
             | Name: /health/alex/steps/raw/<today>10:11                         |
             +-------------------------------------------------------------------+
             | MetaInfo:                                                         |
             | +---------------------------------------------------------------+ |
             | | ContentType: BLOB                                             | |
             | +---------------------------------------------------------------+ |
             +-------------------------------------------------------------------+
             | Content :                                                         |
             | +---------------------------------------------------------------+ |
             | | <Encrypted sample>                                            | |
             | | ...                                                           | |
             | | CKeyLocator: /health/alex/steps/C-KEY/<for-hour>/<today>10:00 | |
             | +---------------------------------------------------------------+ |
             +-------------------------------------------------------------------+
             | SignatureInfo                                                     |
             +-------------------------------------------------------------------+
             | SignatureValue                                                    |
             +-------------------------------------------------------------------+

---

## Alice

### Assumption

Knows that it belongs to the `/health/alex/steps` and `/health/alex` groups.

Empty key caches.

### Timeline

- On retrieval of a data packet

  - Extract `CKeyLocator` field: `/health/alex/steps/C-KEY/<for-hour>/<today>10:00`
   
  - Construct partial C-KEY names for all groups (`CKeyLocator` + `GroupName`) and try to retrieve encrypted C-KEYs

        /health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/steps/E-KEY
         |
         +--> /health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00

        /health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/E-KEY
         |
         +--> timeout (or NACK?)

  - Pick any of the retrieved encrypted C-KEYs and extract full name of E-KEY:

    - `/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00`

  - Determine full name of `D-KEY`: replace `E-KEY` with `D-KEY` append own public key name:

    * `/health/alex/steps/D-KEY/<from><today>08:00/<until><today>24:00`

  - Construct full name of the encrypted D-KEY: append `<for>/{user's-key-name}`

      * `/health/alex/steps/D-KEY/<from><today>08:00/<until><today>24:00/<for>/alice/<version>`

  - Retrieve D-KEY and decrypt C-KEY with user's private key

  - Cache D-KEY and C-KEY
  
  - Decrypt the sample with C-KEY

---

## Bob

### Assumption

Knows that it belongs to the `/health/alex/steps` group.

Empty key caches.

### Timeline

- On retrieval of a data packet

  - Extract `CKeyLocator` field: `/health/alex/steps/C-KEY/<for-hour>/<today>10:00`
   
  - Construct partial C-KEY names for all groups (`CKeyLocator` + `GroupName`) and try to retrieve encrypted C-KEYs

        /health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/steps/E-KEY
         |
         +--> /health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00

  - Pick any of the retrieved encrypted C-KEYs and extract full name of E-KEY:

    - `/health/alex/steps/E-KEY/<from><today>08:00/<until><today>24:00`

  - Determine full name of `D-KEY`: replace `E-KEY` with `D-KEY` append own public key name:

    * `/health/alex/steps/D-KEY/<from><today>08:00/<until><today>24:00`

  - Construct full name of the encrypted D-KEY: append `<for>/{user's-key-name}`

      * `/health/alex/steps/D-KEY/<from><today>08:00/<until><today>24:00/<for>/bob/<version>`

  - Retrieve D-KEY and decrypt C-KEY with user's private key

  - Cache D-KEY and C-KEY
  
  - Decrypt the sample with C-KEY

---

## Yingdi

### Assumption

Knows that it belongs to the `/health/alex/steps` group.

Empty key caches.

### Timeline

- On retrieval of a data packet

  - Extract `CKeyLocator` field: `/health/alex/steps/C-KEY/<for-hour>/<today>10:00`
   
  - Construct partial C-KEY names for all groups (`CKeyLocator` + `GroupName`) and try to retrieve encrypted C-KEYs

        /health/alex/steps/C-KEY/<for-hour>/<today>10:00/<for>/health/alex/steps/E-KEY
         |
         +--> timeout (or NACK?)

  - Declare failure (unauthorized access)
