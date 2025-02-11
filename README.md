# jwks-rs
[![Version](https://img.shields.io/crates/v/jwks-rs)](https://crates.io/crates/jwks-rs)
[![License](https://img.shields.io/crates/l/jwks-rs)](License)

## Installation
```toml
[dependencies]
jwks-rs = "0.1.0"
```

### Usage examples

From a jwks url.
```rust
let jwks_url = "https://www.googleapis.com/oauth2/v3/certs";
let jwks = Jwks::from_jwks_url(jwks_url).await.unwrap();
```

From a openid config url.
```rust
let openid_config_url = "https://accounts.google.com/.well-known/openid-configuration";
let jwks = Jwks::from_oidc_url(openid_config_url).await.unwrap();
```

Use with [jsonwebtokn](https://github.com/Keats/jsonwebtoken) to validate a jwt.
```rust
use jsonwebtoken::{decode, decode_header, TokenData, Validation};
use jwks::Jwks;
use serde::{Deserialize, Serialize};

#[derive(Clone, Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
}

#[tokio::main]
async fn main() {
    let jwt = "...base64-encoded-jwt...";

    // get the kid from jwt
    let header = decode_header(jwt).expect("jwt header should be decoded");
    let kid = header.kid.as_ref().expect("jwt header should have a kid");

    // get a jwk from jwks by kid
    let jwks_url = "https://www.googleapis.com/oauth2/v3/certs";
    let jwks = Jwks::from_jwks_url(jwks_url).await.unwrap();
    let jwk = jwks.keys.get(kid).expect("jwt refer to a unknown key id");

    let validation = Validation::default();
    let decoded_token: TokenData<Claims> =
        decode::<Claims>(jwt, &jwk.decoding_key, &validation).expect("jwt should be valid");
}
```
