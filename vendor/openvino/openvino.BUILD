load("@rules_cc//cc:defs.bzl", "cc_library")
load("@com_intel_plaidml//bzl:template.bzl", "template_rule")

package(default_visibility = ["//visibility:public"])

TAGS = [
    "skip_macos",
    "skip_windows",
]

cc_library(
    name = "benchmark_app",
    srcs = glob([
        "inference-engine/samples/benchmark_app/**/*.hpp",
        "inference-engine/samples/common/**/*.cpp",  # TODO
        "inference-engine/samples/common/**/*.hpp",  # TODO
        "inference-engine/samples/benchmark_app/**/*.cpp",
    ]),
    data = [":plugins"],
    includes = [
        "inference-engine/samples/common",
        "inference-engine/samples/common/format_reader",
    ],
    linkopts = select({
        "@bazel_tools//src/conditions:windows": [],
        "@bazel_tools//src/conditions:darwin_x86_64": [],
        "//conditions:default": [
            "-pthread",
            "-lm",
            "-ldl",
        ],
    }),
    linkstatic = 0,
    deps = [
        ":inference_engine",
        ":mkldnn_plugin",
        "@gflags",
    ],
)

cc_library(
    name = "testing",
    srcs = glob([
        "inference-engine/tests/helpers/*common*cpp",
    ]),
    hdrs = glob([
        "inference-engine/tests/helpers/*common*hpp",
    ]),
    defines = [
        "DATA_PATH=NULL",
    ],
    includes = [
        "inference-engine/tests/helpers",
    ],
    deps = [
        ":inference_engine",
    ],
)

cc_library(
    name = "smoke_tests",
    srcs = [
        "inference-engine/tests/unit/engines/mkldnn/dump_test.cpp",
    ],
    hdrs = [
        "inference-engine/tests/unit/engines/mkldnn/graph/test_graph.hpp",
    ],
    data = [":plugins"],
    includes = [
        "inference-engine/tests/unit/engines/mkldnn/graph",
    ],
    deps = [
        ":mkldnn_plugin",
        ":testing",
        "@gmock//:gtest",
    ],
)

genrule(
    name = "plugins",
    outs = ["plugins.xml"],
    cmd = "echo \"<ie><plugins><plugin name=\\\"CPU\\\" location=\\\"./libmkldnn.so\\\"></plugin></plugins></ie>\" > $@",
)

template_rule(
    name = "mkldnn_version",
    src = "inference-engine/thirdparty/mkl-dnn/include/mkldnn_version.h.in",
    out = "inference-engine/thirdparty/mkl-dnn/include/mkldnn_version.h",
    substitutions = {
        "@MKLDNN_VERSION_MAJOR@": "1",
        "@MKLDNN_VERSION_MINOR@": "1",
        "@MKLDNN_VERSION_PATCH@": "1",
        "@MKLDNN_VERSION_HASH@": "afd",
    },
)

cc_library(
    name = "mkldnn_plugin",
    srcs = glob(
        [
            "inference-engine/thirdparty/mkl-dnn/src/**/*pp",
            "inference-engine/src/mkldnn_plugin/**/*pp",
        ],
        exclude = [
            "inference-engine/src/mkldnn_plugin/mkldnn/os/**/*.cpp",
            "inference-engine/src/mkldnn_plugin/nodes/ext_convert.cpp",
        ],
    ) + select({
        "@bazel_tools//src/conditions:darwin_x86_64": [],
        "@bazel_tools//src/conditions:windows": glob([
            "inference-engine/src/mkldnn_plugin/mkldnn/os/win/*.cpp",
        ]),
        "//conditions:default": glob([
            "inference-engine/src/mkldnn_plugin/mkldnn/os/lin/*.cpp",
        ]),
    }),
    hdrs = glob([
        "inference-engine/thirdparty/mkl-dnn/include/*",
        "inference-engine/thirdparty/mkl-dnn/src/common/*.hpp",
        "inference-engine/thirdparty/mkl-dnn/src/cpu/**/*.h*",
        "inference-engine/thirdparty/mkl-dnn/src/*.hpp",
    ]) + [":mkldnn_version"],
    includes = [
        "inference-engine/src/mkldnn_plugin",
        "inference-engine/src/mkldnn_plugin/mkldnn",
        "inference-engine/thirdparty/mkl-dnn/include",
        "inference-engine/thirdparty/mkl-dnn/src",
        "inference-engine/thirdparty/mkl-dnn/src/common",
        "inference-engine/thirdparty/mkl-dnn/src/cpu",
    ],
    local_defines = [
        "COMPILED_CPU_MKLDNN_QUANTIZE_NODE",
        "COMPILED_CPU_MKLDNN_ACTIVATION_NODE",
    ],
    deps = [":inference_engine"],
    alwayslink = 1,
)

cc_library(
    name = "inc",
    hdrs = glob([
        "inference-engine/include/**/*.h",
        "inference-engine/include/**/*.hpp",
    ]),
    defines = [
        "CI_BUILD_NUMBER=\\\"0\\\"",
        "IE_BUILD_POSTFIX=\\\"\\\"",
    ],
    includes = [
        "inference-engine/include",
    ],
    tags = TAGS,
)

cc_library(
    name = "shared_plugin_tests",
    srcs = glob(
        ["inference-engine/tests/functional/plugin/shared/src/single_layer_tests/*.cpp"],
        exclude = [
            "inference-engine/tests/functional/plugin/shared/src/single_layer_tests/ctc_greedy_decoder.cpp",
            "inference-engine/tests/functional/plugin/shared/src/single_layer_tests/cum_sum.cpp",
            "inference-engine/tests/functional/plugin/shared/src/single_layer_tests/extract_image_patches.cpp",
            "inference-engine/tests/functional/plugin/shared/src/single_layer_tests/nonzero.cpp",
            "inference-engine/tests/functional/plugin/shared/src/single_layer_tests/prior_box_clustered.cpp",
            "inference-engine/tests/functional/plugin/shared/src/single_layer_tests/proposal.cpp",
        ],
    ),
    hdrs = glob(
        ["inference-engine/tests/functional/plugin/shared/include/single_layer_tests/*.hpp"],
        exclude = [
            "inference-engine/tests/functional/plugin/shared/include/single_layer_tests/ctc_greedy_decoder.hpp",
            "inference-engine/tests/functional/plugin/shared/include/single_layer_tests/cum_sum.hpp",
            "inference-engine/tests/functional/plugin/shared/include/single_layer_tests/extract_image_patches.hpp",
            "inference-engine/tests/functional/plugin/shared/include/single_layer_tests/nonzero.hpp",
            "inference-engine/tests/functional/plugin/shared/include/single_layer_tests/prior_box_clustered.hpp",
            "inference-engine/tests/functional/plugin/shared/include/single_layer_tests/proposal.hpp",
        ],
    ),
    includes = ["inference-engine/tests/functional/plugin/shared/include"],
    deps = [
        ":functional_test_utils",
        ":inference_engine",
        ":ngraph_function_tests",
        "@com_google_googletest//:gtest",
    ],
)

cc_library(
    name = "functional_test_utils",
    srcs = glob(["inference-engine/tests/ie_test_utils/functional_test_utils/*.cpp"]),
    hdrs = glob(
        ["inference-engine/tests/ie_test_utils/functional_test_utils/*.hpp"],
    ),
    includes = ["inference-engine/tests/ie_test_utils"],
    deps = [
        ":inference_engine",
        ":ngraph_function_tests",
        "@com_google_googletest//:gtest",
    ],
)

cc_library(
    name = "ngraph_function_tests",
    srcs = glob([
        "inference-engine/tests/ngraph_functions/src/*.cpp",
        "inference-engine/tests/ngraph_functions/src/utils/*.cpp",
    ]),
    hdrs = glob([
        "inference-engine/tests/ngraph_functions/include/ngraph_functions/*.hpp",
        "inference-engine/tests/ngraph_functions/include/ngraph_functions/utils/*.hpp",
    ]),
    includes = [
        "inference-engine/tests/ngraph_functions/include/",
    ],
    deps = [
        ":inference_engine",
    ],
)

cc_library(
    name = "common_test_utils",
    srcs = glob(["inference-engine/tests/ie_test_utils/common_test_utils/*.cpp"]),
    hdrs = glob(["inference-engine/tests/ie_test_utils/common_test_utils/*.hpp"]),
    copts = ["-w"],
    includes = ["inference-engine/tests/ie_test_utils"],
    deps = [
        ":inference_engine",
        "@com_google_googletest//:gtest",
    ],
)

cc_library(
    name = "legacy_api",
    srcs = glob([
        "inference-engine/src/legacy_api/src/**/*.cpp",
        "inference-engine/src/legacy_api/src/**/*.hpp",
        "inference-engine/src/legacy_api/src/**/*.h",
    ]),
    hdrs = glob([
        "inference-engine/src/legacy_api/include/**/*.hpp",
    ]),
    copts = [
        "-w",
    ],
    includes = [
        "inference-engine/src/inference_engine",  # TODO: Why does this work?
        "inference-engine/src/legacy_api/include",
        "inference-engine/src/legacy_api/src",
    ],
    tags = TAGS,
    deps = [
        ":inc",
        ":ngraph",
        ":plugin_api",
        ":pugixml",
        ":transformations",
        "@tbb",
    ],
)

cc_library(
    name = "low_precision_transformations",
    srcs = glob(["inference-engine/src/low_precision_transformations/src/**/*.cpp"]),
    copts = ["-w"],
    includes = ["inference-engine/src/low_precision_transformations/include"],
    tags = TAGS,
    deps = [
        ":inc",
        ":legacy_api",
    ],
)

cc_library(
    name = "plugin_api",
    hdrs = glob([
        "inference-engine/src/plugin_api/**/*.h",
        "inference-engine/src/plugin_api/**/*.hpp",
    ]),
    includes = ["inference-engine/src/plugin_api"],
    tags = TAGS,
)

cc_library(
    name = "preprocessing",
    srcs = glob(["inference-engine/src/preprocessing/*.cpp"]),
    copts = ["-w"],
    includes = ["inference-engine/src/preprocessing"],
    tags = TAGS,
    deps = [
        ":fluid_gapi",
        ":inc",
        ":plugin_api",
        "@tbb",
    ],
)

cc_library(
    name = "transformations",
    srcs = glob(["inference-engine/src/transformations/src/**/*.cpp"]),
    copts = ["-w"],
    includes = ["inference-engine/src/transformations/include"],
    tags = TAGS,
    deps = [
        ":inc",
        ":ngraph",
    ],
)

# TODO
cc_library(
    name = "ie_reader",
    srcs = glob([
        "inference-engine/src/readers/ir_reader/*.cpp",
        # TODO
        "inference-engine/src/inference_engine/blob_factory.cpp",
        "inference-engine/src/inference_engine/cnn_network_ngraph_impl.cpp",
        "inference-engine/src/inference_engine/generic_ie.cpp",
        "inference-engine/src/inference_engine/ie_blob_common.cpp",
        "inference-engine/src/inference_engine/ie_data.cpp",
        "inference-engine/src/inference_engine/ie_layouts.cpp",
        "inference-engine/src/inference_engine/ie_memcpy.cpp",
        "inference-engine/src/inference_engine/ie_rtti.cpp",
        "inference-engine/src/inference_engine/network_serializer.cpp",
        "inference-engine/src/inference_engine/precision_utils.cpp",
        "inference-engine/src/inference_engine/system_allocator.cpp",
        "inference-engine/src/inference_engine/xml_parse_utils.cpp",
        # "inference-engine/src/inference_engine/*.cpp",
    ]),
    hdrs = glob([
        "inference-engine/src/readers/ir_reader/*.hpp",
        # TODO
        "inference-engine/include/details/ie_exception.hpp",
    ]),
    includes = [
        # "inference-engine/src/inference_engine",
        "inference-engine/src/readers/ir_reader",  # TODO: Why does this work?
        "inference-engine/src/readers/reader_api",  # TODO: Why does this work?
    ],
    local_defines = [
        "ENABLE_IR_READER",
    ],
    deps = [
        # ":inference_engine",
        ":inc",
        ":legacy_api",
        ":low_precision_transformations",
        ":ngraph",
        ":plugin_api",
        ":preprocessing",
        ":pugixml",
        ":transformations",
        "@tbb",
    ],
    alwayslink = 1,
)

cc_library(
    name = "inference_engine",
    srcs = glob([
        "inference-engine/src/inference_engine/*.cpp",
        "inference-engine/src/inference_engine/threading/*.cpp",
    ]) + select({
        "@bazel_tools//src/conditions:windows": glob([
            "inference-engine/src/inference_engine/os/win/*.cpp",
        ]),
        "@bazel_tools//src/conditions:darwin_x86_64": [],
        "//conditions:default": glob([
            "inference-engine/src/inference_engine/os/lin/*.cpp",
        ]),
    }),
    hdrs = glob([
        # "inference-engine/src/readers/ir_reader/ie_ir_version.hpp",  # TODO: Ok to remove?
    ]),
    copts = ["-w"],
    includes = [
        "inference-engine/src/inference_engine",
        "inference-engine/src/readers/ir_reader",  # TODO: Why does this work?
        "inference-engine/src/readers/reader_api",  # TODO: Why does this work?
    ],
    local_defines = [
        "ENABLE_IR_READER",
    ],
    tags = TAGS,
    deps = [
        ":inc",
        ":legacy_api",
        ":low_precision_transformations",
        ":ngraph",
        ":plugin_api",
        ":preprocessing",
        ":pugixml",
        ":transformations",
        "@tbb",
    ],
    alwayslink = 1,
)

cc_library(
    name = "fluid_gapi",
    srcs = glob([
        "inference-engine/thirdparty/fluid/modules/gapi/src/api/g*.cpp",
        "inference-engine/thirdparty/fluid/modules/gapi/src/compiler/*.cpp",
        "inference-engine/thirdparty/fluid/modules/gapi/src/compiler/passes/*.cpp",
        "inference-engine/thirdparty/fluid/modules/gapi/src/executor/*.cpp",
        "inference-engine/thirdparty/fluid/modules/gapi/src/backends/common/*.cpp",
        "inference-engine/thirdparty/fluid/modules/gapi/src/backends/fluid/*.cpp",
        "inference-engine/thirdparty/fluid/modules/gapi/src/*.hpp",
    ]),
    hdrs = glob([
        "inference-engine/thirdparty/fluid/modules/gapi/include/opencv2/**/*.hpp",
    ]),
    copts = ["-w"],
    defines = [
        "GAPI_STANDALONE",
    ],
    includes = [
        "inference-engine/thirdparty/fluid/modules/gapi/include",
        "inference-engine/thirdparty/fluid/modules/gapi/src",
    ],
    tags = TAGS,
    deps = ["@ade"],
)

cc_library(
    name = "pugixml",
    srcs = glob([
        "inference-engine/thirdparty/pugixml/src/*.cpp",
    ]),
    hdrs = glob([
        "inference-engine/thirdparty/pugixml/src/*.hpp",
    ]),
    copts = ["-w"],
    includes = [
        "inference-engine/thirdparty/pugixml/src",
    ],
    strip_include_prefix = "inference-engine/thirdparty/pugixml/src",
)

cc_library(
    name = "ngraph",
    srcs = glob(
        [
            "ngraph/src/ngraph/*.cpp",
            "ngraph/src/ngraph/*.hpp",
            "ngraph/src/ngraph/builder/*.cpp",
            "ngraph/src/ngraph/descriptor/**/*.cpp",
            "ngraph/src/ngraph/distributed/*.cpp",
            "ngraph/src/ngraph/op/*.cpp",
            "ngraph/src/ngraph/op/**/*.cpp",
            "ngraph/src/ngraph/opsets/*.cpp",
            "ngraph/src/ngraph/pass/*.cpp",
            "ngraph/src/ngraph/pattern/**/*.cpp",
            "ngraph/src/ngraph/runtime/*.cpp",
            "ngraph/src/ngraph/runtime/reference/*.cpp",
            "ngraph/src/ngraph/state/*.cpp",
            "ngraph/src/ngraph/type/*.cpp",
            "ngraph/test/runtime/*.cpp",
            "ngraph/test/runtime/**/*.cpp",
        ],
        exclude = [
            "ngraph/src/ngraph/serializer.cpp",
            "ngraph/test/runtime/ie/*.cpp",
        ],
    ),
    hdrs = glob(
        [
            "ngraph/core/include/ngraph/*.hpp",
            "ngraph/src/ngraph/*.hpp",
            "ngraph/src/ngraph/builder/*.hpp",
            "ngraph/src/ngraph/descriptor/**/*.hpp",
            "ngraph/src/ngraph/distributed/*.hpp",
            "ngraph/src/ngraph/op/*.hpp",
            "ngraph/src/ngraph/op/**/*.hpp",
            "ngraph/src/ngraph/opsets/*.hpp",
            "ngraph/src/ngraph/pass/*.hpp",
            "ngraph/src/ngraph/pattern/*.hpp",
            "ngraph/src/ngraph/runtime/**/*.hpp",
            "ngraph/src/ngraph/state/*.hpp",
            "ngraph/src/ngraph/pattern/**/*.hpp",
            "ngraph/src/ngraph/type/*.hpp",
            "ngraph/test/runtime/*.hpp",
            "ngraph/test/runtime/**/*.hpp",
        ],
        exclude = [
            "ngraph/test/runtime/ie/*.hpp",
        ],
    ),
    defines = [
        "NGRAPH_JSON_DISABLE",
        "NGRAPH_INTERPRETER_ENABLE",
        "NGRAPH_VERSION=\\\"0.21.0\\\"",
    ],
    includes = [
        "ngraph/core/include",
        "ngraph/src",
        "ngraph/src/ngraph",
        "ngraph/test/runtime",
    ],
    local_defines = [
        "PROJECT_ROOT_DIR=\\\"./\\\"",
        "SHARED_LIB_PREFIX=\\\"\\\"",
        "SHARED_LIB_SUFFIX=\\\"\\\"",
    ],
    tags = TAGS,
)
