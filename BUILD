#
# Copyright (C) 2021 Vaticle
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU Affero General Public License as
# published by the Free Software Foundation, either version 3 of the
# License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Affero General Public License for more details.
#
# You should have received a copy of the GNU Affero General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.
#

load("//:deployment.bzl", deployment_github = "deployment")
load("//:rules.bzl", "jvm_application_image")
load("@io_bazel_rules_kotlin//kotlin:kotlin.bzl", "kt_jvm_binary", "kt_jvm_library")
load("@rules_pkg//:pkg.bzl", "pkg_zip")
load("@vaticle_dependencies//distribution:deployment.bzl", "deployment")
load("@vaticle_dependencies//tool/checkstyle:rules.bzl", "checkstyle_test")
load("@vaticle_bazel_distribution//common:rules.bzl", "checksum", "assemble_targz", "assemble_zip", "java_deps", "assemble_versioned")
load("@vaticle_bazel_distribution//common/tgz2zip:rules.bzl", "tgz2zip")
load("@vaticle_bazel_distribution//github:rules.bzl", "deploy_github")
load("@vaticle_bazel_distribution//brew:rules.bzl", "deploy_brew")
load("@io_bazel_rules_kotlin//kotlin/internal:toolchains.bzl", "define_kt_toolchain")

java_binary(
    name = "studio-bin-mac",
    main_class = "com.vaticle.typedb.studio.viewer.Studio",
    runtime_deps = [
        "//viewer:viewer",
        "@maven//:org_jetbrains_skiko_skiko_jvm_runtime_macos_x64",
    ],
    classpath_resources = ["//config/logback:logback-test-xml"],
)

java_binary(
    name = "studio-bin-windows",
    main_class = "com.vaticle.typedb.studio.viewer.Studio",
    runtime_deps = [
        "//viewer:viewer",
        "@maven//:org_jetbrains_skiko_skiko_jvm_runtime_windows_x64",
    ],
    classpath_resources = ["//config/logback:logback-test-xml"],
)

java_binary(
    name = "studio-bin-linux",
    main_class = "com.vaticle.typedb.studio.viewer.Studio",
    runtime_deps = [
        "//viewer:viewer",
        "@maven//:org_jetbrains_skiko_skiko_jvm_runtime_linux_x64",
    ],
    classpath_resources = ["//config/logback:logback-test-xml"],
)

assemble_files = {
    "//config/logback:logback-xml": "logback.xml",
    "//:LICENSE": "LICENSE",
}

kt_jvm_library(
    name = "jvm-application-image-builder-lib",
    srcs = ["JVMApplicationImageBuilder.kt"],
    deps = ["@maven//:org_zeroturnaround_zt_exec"],
)

java_binary(
    name = "jvm-application-image-builder-bin",
    runtime_deps = [":jvm-application-image-builder-lib"],
    main_class = "com.vaticle.typedb.studio.JVMApplicationImageBuilderKt",
)

# TODO: Create a MANIFEST file in the jvm_binary and read it to determine the main jar and main class
jvm_application_image(
    name = "application-image",
    application_name = "TypeDB Studio",
    icon_mac = "//resources/icons/vaticle:vaticle-bot-mac",
    icon_linux = "//resources/icons/vaticle:vaticle-bot-linux",
    icon_windows = "//resources/icons/vaticle:vaticle-bot-windows",
    filename = "typedb-studio-" + select({
        "@vaticle_dependencies//util/platform:is_mac": "mac",
        "@vaticle_dependencies//util/platform:is_linux": "linux",
        "@vaticle_dependencies//util/platform:is_windows": "windows",
        "//conditions:default": "mac",
    }),
    version_file = ":VERSION",
    jvm_binary = select({
        "@vaticle_dependencies//util/platform:is_mac": ":studio-bin-mac",
        "@vaticle_dependencies//util/platform:is_linux": ":studio-bin-linux",
        "@vaticle_dependencies//util/platform:is_windows": ":studio-bin-windows",
        "//conditions:default": ":studio-bin-mac",
    }),
    main_jar = "com-vaticle-typedb-studio-0.0.0.jar",
    main_class = "com.vaticle.typedb.studio.viewer.Studio",
    deps_use_maven_name = False,
    additional_files = assemble_files,
    mac_entitlements = "//config/mac:entitlements-mac-plist",
    mac_code_signing_cert = "@vaticle_apple_developer_id_application_cert//file",
)

# A little misleading. Because of the way our java_deps target is generated, this will actually produce a Mac runner
# if built on Mac, and fail to produce anything useful if built on Windows.
assemble_targz(
    name = "linux-java-binary",
    targets = [":application-image-deps", "//binary:assemble-bash-targz"],
    additional_files = assemble_files,
    output_filename = "typedb-studio-linux-java-binary",
    visibility = ["//:__pkg__"]
)

deploy_github(
    name = "deploy-github",
    organisation = deployment_github['github.organisation'],
    repository = deployment_github['github.repository'],
    title = "TypeDB Studio",
    title_append_version = True,
    release_description = "//:RELEASE_TEMPLATE.md",
    archive = ":application-image",
#    archive = ":hello-bundle",
    version_file = ":VERSION",
    draft = False
)

deploy_brew(
    name = "deploy-brew",
    snapshot = deployment['brew.snapshot'],
    release = deployment['brew.release'],
    formula = "//config/brew:typedb-studio.rb",
#    checksum = "//:checksum",
    version_file = "//:VERSION",
    type = "cask",
)

checkstyle_test(
    name = "checkstyle",
    include = glob(["*", ".grabl/*", ".circleci/**"]),
    exclude = glob(["docs/*", ".circleci/windows/*"]),
    license_type = "agpl",
)

# CI targets that are not declared in any BUILD file, but are called externally
filegroup(
    name = "ci",
    data = [
        "@vaticle_dependencies//tool/checkstyle:test-coverage",
        "@vaticle_dependencies//tool/release/createnotes:bin",
        "@vaticle_dependencies//tool/bazelrun:rbe",
    ],
)
