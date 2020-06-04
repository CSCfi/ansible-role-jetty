# Ansible Role: Jetty

[![Build Status](https://travis-ci.org/CSCfi/ansible-role-jetty.svg?branch=master)](https://travis-ci.org/CSCfi/ansible-role-jetty)

An role which downloads and unpacks Jetty under /opt. Jetty user and group are also created if needed.

## Requirements

None. Purpose of this role is to perform basic installation with security in mind.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml` for details):

- `jetty_version: 9.4.29.v20200521`
- `jetty_install: true`
- `jetty_force_install: false`
- `system_base: /opt` - base directory where distribution package(s) will be installed to;
- `jetty_home: /opt/jetty` - defines the location of symlink name to the Jetty distribution with its libs, default modules and default XML files (typically start.jar, lib, etc), as well asJetty user home directory. **Important**, it should be treated as a standard of truth and remain unmodified or changed;
- `jetty_base: /var/lib/jetty` - defines the location of a specific implementation of a Jetty server, its configuration, logs and web applications (typically start.d/*.ini files, logs and webapps); all changes or additions to your configuration should take place there;
- `jetty_logs: /var/log/jetty`
- `jetty_dir_mode: 0750`
- `jetty_file_mode: 0640`
- `jetty_user_name: jetty`
- `jetty_group_name: jetty`
- `jetty_group_create: true`
- `jetty_user_create: true`
- `jetty_clean_old: false` - whether to delete any other installed distribution except of the specified version; matching directories by `{{system_base}}/jetty-distribution-*`;
- `jetty_demo_delete: true`
- `jetty_service_enabled: true`
- `jetty_pid: {{ jetty_base }}/jetty.pid"`
- `jetty_start_log: {{ jetty_logs }}/jetty-start.log"`
- `jetty_state: {{ jetty_base }}/jetty.state"`
- `jetty_web_default` - dictionary of modified and changeable settings in `webdefault.xml`; other changed setting is disabled directory listings (`dirAllowed`):

    - `compiler: 11`
    - `development: true`
    - `log_verbosity: "WARNING"`

- `jetty_modules_configure: false` - whether to configure any modules for start in Jetty base directory;

- `jetty_modules` - dictionary for configuring enabled modules, see examples in `defaults/main.yml`, as well as `templates/start.d/` and module definitions for default values;
- `jetty_modules_enabled`:

  ```sh
  - connectionlimit
  - customrequestlog
  - deploy
  - ext
  - gzip
  - http
  - http2
  - http2c
  - https
  - inetaccess
  - jsp
  - jstl
  - lowresources
  - plus
  - resources
  - rewrite
  - security
  - server
  - ssl
  - threadlimit
  - websocket
  ```

- `jetty_modules_config_overrides` - some of the modules require extra configuration in XML which name might be not equal to module's name, e.g. `ssl` module but `ssl-context.xml`:

  ```sh
  jetty-inetaccess.xml
  jetty-rewrite.xml
  jetty-ssl-context.xml
  ```

- `jetty_headers` - see <https://securityheaders.com> for more information:

  ```sh
  Cache-Control: no-store, no-cache, must-revalidate, proxy-revalidate
  Expect-CT: enforce, max-age=86400
  Expires: 0
  Pragma: no-cache
  Referrer-Policy: same-origin
  Surrogate-Control: no-store
  X-Content-Type-Options: nosniff
  X-DNS-Prefetch-Control: off
  X-Download-Options: noopen
  X-Frame-Options: SAMEORIGIN
  X-XSS-Protection: 1; mode=block
  ```

- `jetty_ssl_generate_cert: false` - whether to generate a self-signed certificate;
- `tls_cipher_suites` - see <https://www.ssllabs.com/ssltest> and <https://github.com/dev-sec/ssl-baseline> for details:

  ```sh
  # TLS 1.3 cipher suites
  TLS_AES_256_GCM_SHA384
  # TLS 1.2 cipher suites
  TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
  TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
  ```

- `tls_protocols`: TLSv1.3 and TLSv1.2 only;

- `java_options` - note, default Java options are compatible with Java 11 or higher; modify the settings for Java 8:

  ```sh
  -server
  -Xms1g
  -Xmx1g
  -XX:MetaspaceSize=256m
  -XX:MaxMetaspaceSize=256m
  -XX:+UseContainerSupport
  -XX:MaxTenuringThreshold=1
  -XX:+UseAES
  -XX:+UseCompressedOops
  -Djava.security.egd=file:/dev/./urandom
  -Dhttps.protocols={{ tls_protocols | join(",") }}
  -XX:+DisableExplicitGC
  -XX:+UnlockExperimentalVMOptions
  -XX:+UseZGC
  -Xlog:gc:file={{ jetty_logs }}/jvm_gc.log:utctime,pid,level,tags:filecount=5,filesize=1024
  -XX:+PrintClassHistogram
  -XX:+HeapDumpOnOutOfMemoryError
  -XX:HeapDumpPath={{ jetty_logs }}/jvm_heapdump.hprof
  ```

## Dependencies

While there is no direct dependencies for the role, it is understandable that all Jetty's requirements, i.e. Java runtime environment, must be satisfied. See [official documentation](https://www.eclipse.org/jetty/documentation/current/what-jetty-version.html) for details.

## Runtime settings

- if to generate a self-signed certificate is disabled then a valid `keystore` file must be provided inside `jetty_base` [sub-]folder.

## Security

The following steps have been taken to secure and harden the service:

- disabled server version and `XPoweredBy` information in HTTP headers;
- HTTP/2, including `h2c`, has been enabled;
- HTTPS is limited to TLS v1.2 and 1.3 with strong ciphers only (see `templates/etc/jetty-ssl-context.xml` or corresponding variables above);
- TLS renegotiation has been disabled to prevent [certain attacks](https://owasp.org/www-pdf-archive/OWASP_-_TLS_Renegotiation_Vulnerability.pdf);
- default HTTP headers have been modified as per above;
- strick order of ciphers is enforced;
- connection limit is set to 1500;
- GZIP compression is enabled;
- low resource monitor has been configured;
- thread limit per remote IP is set to 10;
- servlet standard security handling to the class path has been added;
- logging directory is set to `${JETTY_BASE/logs}`;
- server logging pattern is expanded with full [RFC 3339](https://tools.ietf.org/html/rfc3339) timestamp and other extra fields;
- request logging has been enabled in [expanded NCSA](https://en.wikipedia.org/wiki/Common_Log_Format) format;
- as an example, HTTP header rewrite rule has been configured (`jetty/etc/jetty-rewrite.xml`) and disabled (`jetty/start.d/rewrite.ini-disabled`);
- enforced compliance:
    - [RFC6265](https://tools.ietf.org/html/rfc6265) - the HTTP Cookie and Set-Cookie header fields;
    - [RFC7230](https://tools.ietf.org/html/rfc7230) - HTTP/1.1;
    - [RFC7578](https://tools.ietf.org/html/rfc7578) - the multipart/form-data media type.

## Example Playbook

```yaml
- hosts: all
    roles:
      - role: CSCfi.jetty
        vars:
          jetty_modules_configure: true
```
