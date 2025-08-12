# Test

using System;
using System.Collections.Generic;
using Tizen.NUI;
using Tizen.NUI.BaseComponents;

namespace YogaLayoutReference
{
    class YogaLayoutReferenceDemo : NUIApplication
    {
        private View _root;
        private TextLabel _title;
        private View _stage;
        private List<Scenario> _scenarios;
        private int _index = 0;
        private readonly Color[] Palette =
        {
            new Color(0.91f,0.30f,0.24f,1), // red
            new Color(0.18f,0.80f,0.44f,1), // green
            new Color(0.20f,0.60f,0.86f,1), // blue
            new Color(0.61f,0.35f,0.71f,1), // purple
            new Color(1.00f,0.76f,0.03f,1), // yellow
            new Color(0.20f,0.29f,0.37f,1), // slate
        };

        protected override void OnCreate()
        {
            base.OnCreate();
            InitUI();
            BuildScenarios();
            ApplyScenario(_scenarios[_index]);
        }

        private void InitUI()
        {
            Window.Instance.BackgroundColor = new Color(0.06f, 0.08f, 0.10f, 1.0f);

            _root = new View
            {
                Layout = new LinearLayout
                {
                    LinearOrientation = LinearLayout.Orientation.Vertical,
                    CellPadding = new Size2D(0, 16),
                },
                WidthSpecification = LayoutParamPolicies.MatchParent,
                HeightSpecification = LayoutParamPolicies.MatchParent,
                Padding = new Extents(48, 48, 48, 48),
            };
            Window.Instance.Add(_root);

            _title = new TextLabel
            {
                PointSize = 14.0f,
                TextColor = new Color(1,1,1,0.85f),
                FontFamily = "SamsungOneUI, SamsungOne",
                MultiLine = true,
                Ellipsis = false,
                WidthSpecification = LayoutParamPolicies.MatchParent,
                HeightSpecification = LayoutParamPolicies.WrapContent,
            };
            _root.Add(_title);

            // Stage: 시나리오 컨테이너(카드 느낌)
            _stage = new View
            {
                BackgroundColor = new Color(1,1,1,0.03f),
                CornerRadius = 24.0f,
                WidthSpecification = LayoutParamPolicies.MatchParent,
                HeightSpecification = LayoutParamPolicies.MatchParent,
                Padding = new Extents(24,24,24,24),
            };
            _root.Add(_stage);

            // 좌/우 키: 이전/다음 시나리오, 스페이스: 아이템 리빌드
            Window.Instance.KeyEvent += (s, e) =>
            {
                if (e.Key.State != Key.StateType.Down) return;

                switch (e.Key.KeyPressedName)
                {
                    case "Left":
                        _index = (_index - 1 + _scenarios.Count) % _scenarios.Count;
                        ApplyScenario(_scenarios[_index], animate:true);
                        break;
                    case "Right":
                        _index = (_index + 1) % _scenarios.Count;
                        ApplyScenario(_scenarios[_index], animate:true);
                        break;
                    case "space":
                        // 동일 시나리오를 새 아이템으로 리빌드(랜덤 색/사이즈로 미묘한 변주)
                        ApplyScenario(_scenarios[_index], rebuildItems:true, animate:true);
                        break;
                }
            };
        }

        private void BuildScenarios()
        {
            _scenarios = new List<Scenario>
            {
                // 1) FlexDirection & Wrap
                new Scenario(
                    "Direction: Row | Wrap: Wrap | AlignItems: Center | Justify: SpaceBetween\n" +
                    "여러 크기의 아이템이 가로로 흐르며 줄바꿈. 세로축은 Center, 가로축은 SpaceBetween.",
                    container =>
                    {
                        var layout = new FlexLayout
                        {
                            Direction = FlexLayout.FlexDirection.Row,
                            WrapType = FlexLayout.FlexWrapType.Wrap,
                            Justification = FlexLayout.FlexJustification.SpaceBetween,
                            Alignment = FlexLayout.AlignmentType.Center,
                            AlignContent = FlexLayout.AlignmentType.Stretch,
                            ItemsPadding = new Size2D(16, 16)
                        };
                        container.Layout = layout;
                        return Items(
                            new ItemSpec(160, 96), new ItemSpec(120, 120), new ItemSpec(220, 80),
                            new ItemSpec(100, 100), new ItemSpec(140, 160), new ItemSpec(180, 120));
                    }),

                // 2) Column + AlignItems / Justify variations
                new Scenario(
                    "Direction: Column | Wrap: NoWrap | AlignItems: Stretch | Justify: SpaceAround\n" +
                    "세로 흐름, 가로폭은 Stretch. 상하 여백을 균등하게.",
                    container =>
                    {
                        var layout = new FlexLayout
                        {
                            Direction = FlexLayout.FlexDirection.Column,
                            WrapType = FlexLayout.FlexWrapType.NoWrap,
                            Justification = FlexLayout.FlexJustification.SpaceAround,
                            Alignment = FlexLayout.AlignmentType.Stretch,
                            ItemsPadding = new Size2D(0, 12)
                        };
                        container.Layout = layout;
                        return Items(
                            new ItemSpec(matchParentWidth:true, height:72),
                            new ItemSpec(matchParentWidth:true, height:120),
                            new ItemSpec(matchParentWidth:true, height:96));
                    }),

                // 3) Grow / Shrink / Basis 효과 비교
                new Scenario(
                    "Grow / Shrink / Basis 비교 (Row, NoWrap)\n" +
                    "아이템: [Grow 2], [Basis 200 + Shrink 0], [Grow 1]",
                    container =>
                    {
                        var layout = new FlexLayout
                        {
                            Direction = FlexLayout.FlexDirection.Row,
                            WrapType = FlexLayout.FlexWrapType.NoWrap,
                            Justification = FlexLayout.FlexJustification.FlexStart,
                            Alignment = FlexLayout.AlignmentType.Stretch,
                            ItemsPadding = new Size2D(12, 0)
                        };
                        container.Layout = layout;

                        var a = MakeCard("Grow 2", 0, 0, 96);
                        FlexLayout.SetFlexGrow(a, 2.0f);

                        var b = MakeCard("Basis 200\nShrink 0", 1, 0, 96);
                        a.Size2D = new Size2D(0, 96);
                        b.Size2D = new Size2D(200, 96); // basis 역할
                        FlexLayout.SetFlexShrink(b, 0.0f);

                        var c = MakeCard("Grow 1", 2, 0, 96);
                        FlexLayout.SetFlexGrow(c, 1.0f);

                        return new[] { a, b, c };
                    }),

                // 4) AlignContent(멀티라인) 비교
                new Scenario(
                    "Wrap: Wrap | AlignContent: SpaceBetween\n" +
                    "멀티라인 컨텐츠의 교차축 분배를 확인.",
                    container =>
                    {
                        var layout = new FlexLayout
                        {
                            Direction = FlexLayout.FlexDirection.Row,
                            WrapType = FlexLayout.FlexWrapType.Wrap,
                            Justification = FlexLayout.FlexJustification.Center,
                            Alignment = FlexLayout.AlignmentType.Center,
                            AlignContent = FlexLayout.AlignmentType.SpaceBetween,
                            ItemsPadding = new Size2D(12, 12)
                        };
                        container.Layout = layout;
                        return Items(
                            new ItemSpec(140, 80), new ItemSpec(140, 120), new ItemSpec(160, 80),
                            new ItemSpec(120, 100), new ItemSpec(180, 80), new ItemSpec(120, 120),
                            new ItemSpec(100, 100), new ItemSpec(160, 80));
                    }),

                // 5) AlignSelf 오버라이드
                new Scenario(
                    "AlignItems: Center + AlignSelf 오버라이드 (start/end/stretch)\n" +
                    "개별 아이템이 부모 정렬을 무시하고 교차축 정렬을 덮어씀.",
                    container =>
                    {
                        var layout = new FlexLayout
                        {
                            Direction = FlexLayout.FlexDirection.Row,
                            WrapType = FlexLayout.FlexWrapType.Wrap,
                            Justification = FlexLayout.FlexJustification.Center,
                            Alignment = FlexLayout.AlignmentType.Center,
                            ItemsPadding = new Size2D(16, 16)
                        };
                        container.Layout = layout;

                        var a = MakeCard("alignSelf: FlexStart", 0, 0, 80);
                        FlexLayout.SetAlignSelf(a, FlexLayout.AlignmentType.FlexStart);

                        var b = MakeCard("alignSelf: Stretch", 1, 0, 80);
                        b.WidthSpecification = LayoutParamPolicies.Fixed;
                        b.HeightSpecification = LayoutParamPolicies.MatchParent;
                        FlexLayout.SetAlignSelf(b, FlexLayout.AlignmentType.Stretch);

                        var c = MakeCard("alignSelf: FlexEnd", 2, 0, 80);
                        FlexLayout.SetAlignSelf(c, FlexLayout.AlignmentType.FlexEnd);

                        var d = MakeCard("inherit (Center)", 3, 0, 80);

                        return new[] { a, b, c, d };
                    }),
            };
        }

        private void ApplyScenario(Scenario s, bool rebuildItems = false, bool animate = false)
        {
            _title.Text = $"Yoga/Flex Reference Test  ·  {(_index+1)}/{_scenarios.Count}\n{DateTime.Now:HH:mm:ss}\n\n{ s.Description }";

            // 컨테이너(카드) 안쪽에 실제 테스트 레이아웃 뷰를 하나 유지
            View testArea = null;
            if (!rebuildItems && _stage.Children.Count > 0)
            {
                testArea = _stage.Children[0];
                testArea.RemoveAll();
            }
            else
            {
                _stage.RemoveAll();
                testArea = new View
                {
                    WidthSpecification = LayoutParamPolicies.MatchParent,
                    HeightSpecification = LayoutParamPolicies.MatchParent,
                    Padding = new Extents(24, 24, 24, 24)
                };
                _stage.Add(testArea);
            }

            // 시나리오가 레이아웃/아이템을 구성
            var items = s.Builder(testArea);

            foreach (var v in items)
            {
                if (animate)
                {
                    v.Opacity = 0.0f;
                    testArea.Add(v);
                    v.AnimateTo("Opacity", 1.0f, 180, Animation.Interpolation.Linear);
                    // 약간의 위치 튕김 효과
                    v.PositionUsesPivotPoint = true;
                    v.PivotPoint = PivotPoint.Center;
                    v.Scale = new Vector3(0.98f, 0.98f, 1);
                    v.AnimateTo("Scale", new Vector3(1, 1, 1), 220, Animation.Interpolation.EaseOut);
                }
                else
                {
                    testArea.Add(v);
                }
            }
        }

        // 아이템 생성 스펙 -> 카드 모양으로 뷰 생성
        private View MakeCard(string title, int colorIndex, int width = 140, int height = 100)
        {
            var bg = new View
            {
                WidthSpecification = width > 0 ? LayoutParamPolicies.Fixed : LayoutParamPolicies.WrapContent,
                HeightSpecification = height > 0 ? LayoutParamPolicies.Fixed : LayoutParamPolicies.WrapContent,
                Size2D = new Size2D(Math.Max(1, width), Math.Max(1, height)),
                BackgroundColor = Palette[colorIndex % Palette.Length].WithAlpha(0.22f),
                CornerRadius = 16.0f,
                Padding = new Extents(16, 16, 16, 16),
            };

            var label = new TextLabel
            {
                Text = title,
                TextColor = new Color(1,1,1,0.9f),
                PointSize = 10.0f,
                MultiLine = true,
                WidthSpecification = LayoutParamPolicies.WrapContent,
                HeightSpecification = LayoutParamPolicies.WrapContent,
            };
            bg.Add(label);
            return bg;
        }

        // 여러 아이템을 간단히 만들어주는 헬퍼
        private View[] Items(params ItemSpec[] specs)
        {
            var list = new List<View>();
            var rnd = new Random();
            for (int i = 0; i < specs.Length; i++)
            {
                var spec = specs[i];
                var idx = i + rnd.Next(0, 2); // 살짝 랜덤 컬러 변주
                var v = MakeCard($"#{i+1}\n{spec}", idx, spec.Width, spec.Height);

                if (spec.MatchParentWidth)
                    v.WidthSpecification = LayoutParamPolicies.MatchParent;
                if (spec.MatchParentHeight)
                    v.HeightSpecification = LayoutParamPolicies.MatchParent;

                if (spec.Grow.HasValue) FlexLayout.SetFlexGrow(v, spec.Grow.Value);
                if (spec.Shrink.HasValue) FlexLayout.SetFlexShrink(v, spec.Shrink.Value);
                if (spec.AlignSelf.HasValue) FlexLayout.SetAlignSelf(v, spec.AlignSelf.Value);

                // 외곽 마진(아이템 간격과 별개)
                if (spec.Margin.HasValue)
                {
                    v.Margin = new Extents((ushort)spec.Margin.Value, (ushort)spec.Margin.Value,
                                           (ushort)spec.Margin.Value, (ushort)spec.Margin.Value);
                }
                list.Add(v);
            }
            return list.ToArray();
        }

        // 실행
        static void Main(string[] args)
        {
            var app = new YogaLayoutReferenceDemo();
            app.Run(args);
        }

        // ------------ 내부 타입들 ------------
        private sealed record Scenario(string Description, Func<View, View[]> Builder);

        private sealed class ItemSpec
        {
            public int Width { get; }
            public int Height { get; }
            public bool MatchParentWidth { get; }
            public bool MatchParentHeight { get; }
            public float? Grow { get; }
            public float? Shrink { get; }
            public FlexLayout.AlignmentType? AlignSelf { get; }
            public int? Margin { get; }

            public ItemSpec(
                int width = 140, int height = 100,
                bool matchParentWidth = false, bool matchParentHeight = false,
                float? grow = null, float? shrink = null,
                FlexLayout.AlignmentType? alignSelf = null,
                int? margin = null)
            {
                Width = width; Height = height;
                MatchParentWidth = matchParentWidth; MatchParentHeight = matchParentHeight;
                Grow = grow; Shrink = shrink; AlignSelf = alignSelf; Margin = margin;
            }

            public override string ToString()
            {
                string size = $"{(MatchParentWidth ? "match" : Width)}×{(MatchParentHeight ? "match" : Height)}";
                string gs = (Grow.HasValue ? $" grow:{Grow}" : "") + (Shrink.HasValue ? $" shrink:{Shrink}" : "");
                string asf = AlignSelf.HasValue ? $" alignSelf:{AlignSelf}" : "";
                return $"{size}{gs}{asf}";
            }
        }
    }

    // 간결한 색상 유틸
    static class ColorExt
    {
        public static Color WithAlpha(this Color c, float a) => new Color(c.R, c.G, c.B, a);
    }
}